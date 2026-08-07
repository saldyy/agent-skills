---
name: concurrency
description: Goroutine lifetimes, channels vs mutexes, atomics in Go
metadata:
  tags: goroutines, channels, mutex, atomics, concurrency
---

# Concurrency in Go

## Never Start a Goroutine Without a Stop Mechanism

A goroutine blocked on a channel send/receive is not garbage collected even if nothing else references that channel — it leaks for the life of the process. Before writing `go func() { ... }()`, know how it ends:

1. Every goroutine needs a predictable end time, a cancellation signal, or both.
2. The caller must be able to wait for it to finish.
3. Don't start goroutines from `init()` — expose a lifecycle method (`Start`/`Stop`/`Close`) instead so callers control it.
4. Keep synchronization scoped to a function; factor the logic into a plain synchronous function and let the caller decide whether to run it concurrently.

```go
// Good - explicit lifetime (Go 1.25+ WaitGroup.Go)
var wg sync.WaitGroup
for _, item := range queue {
    wg.Go(func() { process(ctx, item) })
}
wg.Wait()
```

```go
// Bad - no way to stop or wait
go func() {
    for {
        flush()
        time.Sleep(delay)
    }
}()
```

On Go versions before 1.25, use the explicit `wg.Add(1)` / `defer wg.Done()` pattern inside the closure instead of `wg.Go`.

Use [`go.uber.org/goleak`](https://pkg.go.dev/go.uber.org/goleak) in tests to catch leaks before they reach production.

## Prefer Synchronous Functions

Write the function synchronously and let the *caller* add concurrency if it needs to. A synchronous function is easier to test (call it, check the result — no polling), easier to reason about (no goroutine lifetime to track), and easier to compose. It's usually much harder to remove unwanted concurrency from a function's public API than to add it at the call site.

## Share by Communicating

Go's default is: don't communicate by sharing memory, share memory by communicating.

- **Channels** — use when one goroutine produces a value and another consumes it, or to signal completion/cancellation.
- **Mutex** (`sync.Mutex` / `sync.RWMutex`) — use when the problem is really "protect this shared data structure" (a cache, a counter) and channels would just add ceremony.

Default to channels; reach for a mutex when the shared-state framing is more natural than a producer/consumer framing.

## Mutex Details

The zero value of `sync.Mutex` is ready to use — you almost never need `new(sync.Mutex)` or a pointer to one:

```go
// Good
var mu sync.Mutex

// Unnecessary
mu := new(sync.Mutex)
```

Don't embed a mutex in a struct — give it a named `mu` field so `Lock`/`Unlock` stay implementation details rather than leaking into the exported API.

## Channel Direction and Size

Declare channel parameters as send-only or receive-only wherever possible — the compiler then catches misuse (e.g. closing a receive-only channel) and the signature documents ownership:

```go
func produce(out chan<- int)                 { /* send-only */ }
func consume(in <-chan int)                  { /* receive-only */ }
func transform(in <-chan int, out chan<- int) { /* both */ }
```

Keep channel buffer size at zero (unbuffered) or one. Any larger size needs a comment justifying: how the number was chosen, what bounds it under load, and what happens when a sender blocks.

## Atomics

Use the typed atomics (`atomic.Bool`, `atomic.Int64`, ... — stdlib since Go 1.19) instead of raw `int32`/`int64` fields with manual `atomic.*` calls. A raw field makes it trivially easy to forget the atomic accessor on one code path and introduce a race:

```go
// Good - type-safe, can't accidentally do a non-atomic read
var running atomic.Bool
running.Store(true)
if running.Load() { ... }

// Bad - easy to forget atomic access somewhere
var running int32
atomic.StoreInt32(&running, 1)
if running == 1 { ... } // not atomic - race
```

## Document Thread-Safety When It's Not Obvious

Callers assume read-only operations are safe for concurrent use and mutating ones are not. Call out the exception explicitly when:

- A method that looks read-only actually mutates internal state (e.g. an LRU's `Get` touching recency data).
- A type is deliberately safe for concurrent use (document it on the type, not just a method).
- An interface has a concurrency contract implementations must honor.
