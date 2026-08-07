---
name: data-structures
description: Slices, maps, arrays, and copying semantics in Go
metadata:
  tags: slices, maps, arrays, append, copy
---

# Data Structures in Go

## Choosing One

```
Ordered collection?
├─ Fixed size, known at compile time → array [N]T
└─ Dynamic size                      → slice []T
   ├─ Approximate size known?  → make([]T, 0, capacity)
   └─ Unknown, or needs to serialize to JSON `null` when empty? → var s []T (nil)

Key-value lookup?
└─ map[K]V
   ├─ Approximate size known? → make(map[K]V, capacity)
   └─ Only need membership?   → map[T]struct{} (a set)
```

## Slices Are a View, Not a Container

A slice is a three-word header — pointer, length, capacity — describing a window into an underlying array. Slicing doesn't copy data:

```go
data := [4]int{1, 2, 3, 4}
a := data[0:2] // [1, 2]
b := data[1:3] // [2, 3]

b[0] = 99
fmt.Println(a)    // [1, 99]  - a and b share storage with data
fmt.Println(data) // [1, 99, 3, 4]
```

Because the header is passed by value, `append` inside a function can't change the caller's header — only the elements it can reach. Always assign the result of `append`:

```go
x := []int{1, 2, 3}
x = append(x, 4, 5, 6)
x = append(x, y...) // appending a slice - note the `...`
```

When capacity is exceeded, `append` allocates a new backing array and the returned slice points at it; the caller's original variable still points at the old array until reassigned. That's why the return value matters, not just the mutation.

## Slice Gotchas

**Aliasing through a shared backing array** — mutating a sub-slice mutates the original:

```go
original := []int{1, 2, 3, 4, 5}
subset := original[1:3]
subset[0] = 99
fmt.Println(original) // [1, 99, 3, 4, 5]

// Fix: copy out if independence is required
subset := make([]int, 2)
copy(subset, original[1:3])
```

**Pinning a large array through a small slice** — a small slice into a large backing array keeps the whole array alive:

```go
// Bad - keeps the whole file in memory
func getHeader(file []byte) []byte { return file[:100] }

// Good - copy releases the large backing array
func getHeader(file []byte) []byte {
    header := make([]byte, 100)
    copy(header, file)
    return header
}
```

**`copy` never reallocates** — it copies `min(len(dst), len(src))` elements and handles overlapping slices correctly.

## nil vs Empty Slice

Prefer `var s []string` (nil) over `s := []string{}` for uninitialized state — both have `len`/`cap` of zero and behave identically with `append`/`range`, but nil is the idiomatic default.

The one place the distinction matters: JSON. A nil slice encodes to `null`; `[]string{}` encodes to `[]`. Use the non-nil form explicitly when the API contract requires an array, not `null`.

When designing your own APIs, don't make callers distinguish nil from empty — treat both as "no elements."

## Sets

Use `map[T]struct{}` rather than `map[T]bool` when the map only tracks membership — the empty struct takes no storage and makes the intent explicit:

```go
attended := map[string]struct{}{"Ann": {}, "Joe": {}}
if _, ok := attended[person]; ok {
    fmt.Println(person, "was at the meeting")
}
```

Reach for `map[T]bool` only when `false` carries meaning beyond "absent."

## Copying Structs

Don't copy a value of type `T` if any of its methods are defined on `*T` — copying detaches the value from state the pointer methods expect to mutate in place. This applies to `sync.Mutex`, `sync.WaitGroup`, `bytes.Buffer`, and any struct embedding them:

```go
// Bad - copies the mutex, breaks the lock's invariant
var mu sync.Mutex
mu2 := mu

// Good - pass by pointer
func increment(sc *SafeCounter) {
    sc.mu.Lock()
    sc.count++
    sc.mu.Unlock()
}
```

This is also why you copy at API boundaries: if a caller might hold onto or mutate a slice/map/struct you were handed, copy it in rather than aliasing their storage.
