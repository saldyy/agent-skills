---
name: golang
description: Provides domain-specific best practices for Go development, covering error handling (wrapping with %w vs %v, sentinel vs typed errors), concurrency (goroutine lifetimes, channels vs mutexes), data structures (slices, maps, nil vs empty), testing (table-driven tests, t.Helper, cmp.Diff), naming conventions (MixedCaps, receivers, initialisms), and core style (gofmt, nesting, naked returns). Use when writing or reviewing Go code, deciding how to structure error returns, choosing between a mutex and a channel, writing a Go test, naming a Go identifier, or when the user mentions 'idiomatic Go', 'goroutine leak', 'table-driven test', 'error wrapping', or asks for a Go code review.
metadata:
  tags: go, golang, error-handling, concurrency, testing, style
---

## When to use

Use this skill whenever you are writing, reviewing, or reasoning about Go code, to apply idiomatic patterns drawn from Effective Go, the Google Go Style Guide, the Uber Go Style Guide, and the Go wiki's code review comments.

## Core Principle

Go rewards explicitness over cleverness. When several idioms would work, prefer the one a reviewer can verify at a glance: early returns over nested conditionals, wrapped errors over swallowed ones, channels for handing off ownership, mutexes for guarding state in place.

## Common Workflows

**Returning an error from a new function**: Is this a system boundary (RPC/storage)? → wrap with `%v` → otherwise does the caller need to match it? → sentinel/typed error wrapped with `%w` → otherwise plain `fmt.Errorf("...: %w", err)`. See [rules/error-handling.md](rules/error-handling.md).

**Adding concurrency**: Default to a synchronous function and let the caller decide whether to add a goroutine. If a goroutine is genuinely needed, give it an explicit stop mechanism before writing the spawn line. See [rules/concurrency.md](rules/concurrency.md).

**Writing a test**: Do all cases share the same code path? → table-driven test with `t.Helper()`-based helpers and `cmp.Diff` for structural comparisons → otherwise separate test functions per case. See [rules/testing.md](rules/testing.md).

**Naming something new**: What is it — package, interface, receiver, constant, exported func, variable? Walk the decision flow in [rules/naming.md](rules/naming.md) rather than guessing from habits carried over from another language.

**Choosing a data structure**: Ordered + dynamic size → slice (`make` with a capacity hint if the size is known); key-value → map (`map[K]struct{}` for a set); passing to a function that might mutate → copy at the boundary. See [rules/data-structures.md](rules/data-structures.md).

**Formatting/structure questions not covered by a more specific rule**: fall back to [rules/style.md](rules/style.md) — nesting, naked returns, brace placement, `gofmt`.

## How to use

Read individual rule files for detailed explanations and code examples:

- [rules/error-handling.md](rules/error-handling.md) - Error strategy, wrapping, sentinel vs typed errors
- [rules/concurrency.md](rules/concurrency.md) - Goroutine lifetimes, channels vs mutexes, atomics
- [rules/data-structures.md](rules/data-structures.md) - Slices, maps, arrays, copying semantics
- [rules/testing.md](rules/testing.md) - Table-driven tests, helpers, assertions
- [rules/naming.md](rules/naming.md) - Identifier naming decision flow
- [rules/style.md](rules/style.md) - Formatting, nesting, naked returns, core principles
