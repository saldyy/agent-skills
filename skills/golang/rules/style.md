---
name: style
description: Formatting, nesting, and core style principles for Go
metadata:
  tags: style, gofmt, nesting, naked-returns, formatting
---

# Core Style in Go

This is the fallback rule: apply it when a style question isn't covered by a more specific rule ([error-handling.md](error-handling.md), [naming.md](naming.md), [testing.md](testing.md), [concurrency.md](concurrency.md), [data-structures.md](data-structures.md)).

## Priority Order

When two idioms conflict, resolve in this order:

1. **Clarity** — can a reader understand it without extra context?
2. **Simplicity** — is this the simplest way to accomplish the goal?
3. **Concision** — does every line earn its place?
4. **Maintainability** — will this be easy to modify later?
5. **Consistency** — does it match the surrounding code and project conventions?

Clarity wins over concision when they disagree — a clever one-liner that requires re-reading is worse than three plain lines.

## Formatting

Run `gofmt` (or `goimports`) — never hand-format. There's no hard line-length limit, but treat ~99 characters as a soft ceiling; when a line runs long, refactor rather than just wrapping it (extract a variable, split the call).

## Reduce Nesting

Handle error/exceptional cases first and return or `continue` early, keeping the main logic unindented — Go code should read top-to-bottom without following deep `if`/`else` chains:

```go
// Bad - happy path buried two levels deep
for _, v := range data {
    if v.F1 == 1 {
        v = process(v)
        if err := v.Call(); err == nil {
            v.Send()
        } else {
            return err
        }
    } else {
        log.Printf("invalid v: %v", v)
    }
}

// Good - guard clause, happy path flat
for _, v := range data {
    if v.F1 != 1 {
        log.Printf("invalid v: %v", v)
        continue
    }

    v = process(v)
    if err := v.Call(); err != nil {
        return err
    }
    v.Send()
}
```

### Prefer Default + Override Over Setting Both Branches

If a variable ends up set in both branches of an `if`, that's usually a default value with an override in disguise:

```go
// Bad
var a int
if b {
    a = 100
} else {
    a = 10
}

// Good
a := 10
if b {
    a = 100
}
```

## Naked Returns

A bare `return` in a function with named results returns those named values:

```go
func minMax(a, b int) (min, max int) {
    if a < b {
        min, max = a, b
    } else {
        min, max = b, a
    }
    return // returns min, max
}
```

Fine in small functions where the whole body is visible at once. Once a function grows past a handful of lines, spell out the return values explicitly — the reader shouldn't have to scroll up to remember what a bare `return` sends back. Don't name result parameters *just* to enable a naked return; name them only when it genuinely documents the signature better.

## Brace Placement Is Not a Style Choice

Go's lexer auto-inserts a semicolon after any line ending in an identifier, literal, or one of `break continue fallthrough return ++ -- ) }`. That means the opening brace of a control structure *must* be on the same line as the keyword, or the code doesn't compile as intended:

```go
// Good
if i < f() {
    g()
}

// Bad - semicolon gets inserted after f(), this doesn't do what it looks like
if i < f()
{
    g()
}
```

Idiomatic Go only writes explicit semicolons in a `for` clause or to put multiple statements on one line — `gofmt` won't let you drift from this anyway.
