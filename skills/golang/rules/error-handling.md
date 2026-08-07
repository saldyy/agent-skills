---
name: error-handling
description: Error strategy, wrapping, sentinel vs typed errors in Go
metadata:
  tags: errors, error-wrapping, sentinel-errors, errors-is-as
---

# Error Handling in Go

> `errors.Is`, `errors.As`, and `%w` wrapping require Go 1.13+.

Errors are ordinary values in Go — created by code, returned up the stack, and consumed by code. Treat the type you return and the way you wrap it as a deliberate API decision, not an afterthought.

## Choosing a Strategy

1. Crossing a system boundary (RPC, storage, an external API response)? → wrap with `%v` so internal details don't leak into the response.
2. Does the caller need to branch on this specific failure? → sentinel or typed error, wrapped with `%w`.
3. Does the caller only need debugging context? → `fmt.Errorf("...: %w", err)`.
4. Leaf function with nothing to add? → return the error unchanged.

Default to wrapping with `%w`, placed at the end of the format string.

## Never Return Concrete Error Types

Returning a concrete pointer type as `error` is a classic trap: a `nil` `*MyError` stored in an `error` interface is a non-nil interface value, so `err != nil` checks pass even when nothing went wrong.

```go
// Bad
func Bad() *os.PathError { /* ... */ }

// Good
func Good() error { /* ... */ }
```

## Error Strings

Lowercase, no trailing punctuation — the message is usually going to be embedded in another error string via wrapping.

```go
// Bad
err := fmt.Errorf("Something bad happened.")

// Good
err := fmt.Errorf("something bad happened")
```

Exception: proper nouns, acronyms, and exported identifiers keep their casing. Text meant for direct display (logs, CLI output, API responses) can be capitalized.

## Choosing Between Sentinels, Typed Errors, and Plain Errors

| Caller needs to match? | Message | Use |
|---|---|---|
| No | static | `errors.New("message")` |
| No | dynamic | `fmt.Errorf("msg: %v", val)` |
| Yes | static | `var ErrNotFound = errors.New("not found")` |
| Yes | needs fields | custom `error` type, matched with `errors.As` |

Only escalate past a plain wrapped error when a caller actually calls `errors.Is`/`errors.As` on the result — don't build a sentinel or typed error speculatively.

## %v vs %w

- `%v` — use at system boundaries, in logs, or anywhere you want to hide the internal error chain from the caller.
- `%w` — use to preserve the chain for `errors.Is`/`errors.As`. Place it at the end of the format string; if the annotation doesn't add information the caller doesn't already have, skip wrapping and return `err` directly.

```go
if err != nil {
    return fmt.Errorf("loading config %q: %w", path, err)
}
```

## Handling an Error Once You Have It

Never discard an error with `_` silently — either handle it, return it, or (rarely) `log.Fatal`/`panic`. If you're intentionally ignoring one, say why:

```go
n, _ := b.Write(p) // never returns a non-nil error for this writer
```

Handle it exactly once:

```
Error encountered?
├─ Caller can act on it?  → return it (wrapped with %w if it adds context)
├─ Top of the call chain? → log and handle
└─ Neither?                → log at the right level, continue
```

Logging *and* returning the same error is the most common review comment here — pick one.

## Avoid In-Band Error Values

Don't overload a return value to double as an error signal (`-1`, `nil`, `""`). Use a second return value instead:

```go
// Bad
func Lookup(key string) int // -1 means missing

// Good
func Lookup(key string) (string, bool)
```

This also gets a compile error instead of a silent bug if someone writes `Parse(Lookup(key))`.

## Error-First Flow

Handle the error case immediately and keep the happy path unindented:

```go
if err != nil {
    return err
}
// happy path continues, un-indented
```

## Concurrent Errors

For fan-out work where any failure should cancel the rest, use [`errgroup`](https://pkg.go.dev/golang.org/x/sync/errgroup) instead of hand-rolled channels:

```go
g, ctx := errgroup.WithContext(ctx)
g.Go(func() error { return task1(ctx) })
g.Go(func() error { return task2(ctx) })
if err := g.Wait(); err != nil {
    return err
}
```

## Testing Errors

Assert on semantics, not message strings:

```go
// Bad - brittle
if err.Error() != "invalid input" { ... }

// Good
if !errors.Is(err, ErrInvalidInput) { ... }
```
