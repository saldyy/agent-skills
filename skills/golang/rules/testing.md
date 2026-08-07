---
name: testing
description: Table-driven tests, helpers, and assertions in Go
metadata:
  tags: testing, table-driven-tests, cmp-diff, t-helper
---

# Testing in Go

> Diff examples use [`github.com/google/go-cmp`](https://pkg.go.dev/github.com/google/go-cmp/cmp).

## Quick Reference

| Situation | Use |
|---|---|
| Reporting a failure, test should keep running | `t.Error`/`t.Errorf` (default) |
| Setup failed, or later assertions depend on this one | `t.Fatal`/`t.Fatalf` |
| Comparing structs/slices/maps/protos | `cmp.Diff` |
| Many cases share identical logic | table-driven test |
| Need per-case filtering, parallelism, or naming | subtests (`t.Run`) |
| Any test helper function | `t.Helper()` as the first line |
| Teardown inside a helper | `t.Cleanup()`, not `defer` |

## Failure Messages Must Be Self-Diagnosing

Every failure should tell the reader the function, the inputs, what happened, and what was expected — without them opening the test source. Format: `Func(%v) = %v, want %v`, with "got" always printed before "want":

```go
// Good
t.Errorf("Add(2, 3) = %d, want %d", got, 5)

// Bad - missing function name and inputs
t.Errorf("got %d, want %d", got, 5)
```

## No Assertion Libraries

Use `cmp.Diff` for structural comparisons instead of an assertion library — it produces a readable diff and doesn't hide what was actually compared:

```go
if diff := cmp.Diff(want, got); diff != "" {
    t.Errorf("GetPost() mismatch (-want +got):\n%s", diff)
}
```

Keep the `(-want +got)` direction label in the message. For protobufs, add `protocmp.Transform()` as a `cmp` option. Compare values semantically — don't compare serialized JSON strings.

## t.Error vs t.Fatal

Default to `t.Error` so a single run surfaces every failure, not just the first. Use `t.Fatal` only when continuing is actually impossible: setup failed (DB connection, file load), or a later assertion depends on an earlier one succeeding (e.g. decoding something you just encoded).

Never call `t.Fatal`/`t.FailNow` from a goroutine other than the test's own goroutine — the test binary doesn't handle that correctly. Use `t.Error` there instead.

## Table-Driven Tests

Use a table when every case runs the same code path with no conditional setup or mocking — a single `wantErr bool` field is fine, but branching logic per case is a sign you want separate test functions instead.

```go
func TestAdd(t *testing.T) {
    tests := []struct {
        name    string
        a, b    int
        want    int
        wantErr bool
    }{
        {name: "positive", a: 2, b: 3, want: 5},
        {name: "overflow", a: math.MaxInt, b: 1, wantErr: true},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := Add(tt.a, tt.b)
            if (err != nil) != tt.wantErr {
                t.Fatalf("Add(%d, %d) error = %v, wantErr %t", tt.a, tt.b, err, tt.wantErr)
            }
            if err == nil && got != tt.want {
                t.Errorf("Add(%d, %d) = %d, want %d", tt.a, tt.b, got, tt.want)
            }
        })
    }
}
```

Use named struct fields once a case spans multiple lines or has several same-type fields — positional fields become unreadable fast. Always include the actual inputs in the failure message; don't identify a failing case only by its table index.

## Test Helpers

Any function that does setup/assertions on behalf of a test must call `t.Helper()` as its first statement (so failures report the caller's line number, not the helper's) and register cleanup with `t.Cleanup()` rather than `defer`:

```go
func setupTestDB(t *testing.T) *sql.DB {
    t.Helper()
    db, err := sql.Open("sqlite3", ":memory:")
    if err != nil {
        t.Fatalf("could not open database: %v", err)
    }
    t.Cleanup(func() { db.Close() })
    return db
}
```

## Test Error Semantics, Not Strings

```go
// Bad - brittle, breaks on any wording change
if err.Error() != "invalid input" { ... }

// Good - tests the contract, not the message
if !errors.Is(err, ErrInvalidInput) { ... }
```

When a table test only needs to know whether an error occurred, not which one, a boolean presence check is fine:

```go
if gotErr := err != nil; gotErr != tt.wantErr {
    t.Errorf("f(%v) error = %v, want error presence = %t", tt.input, err, tt.wantErr)
}
```
