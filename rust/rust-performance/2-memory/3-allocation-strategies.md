---
source_course: "rust-performance"
source_lesson: "rust-perf-allocation-strategies"
---

# Allocation Strategies

## Introduction

Every heap allocation involves a call to the system allocator, which is orders of magnitude slower than stack allocation. Reducing allocations is often the single most impactful optimization in Rust programs.

## Key Concepts

### Stack vs Heap

```rust
let x: i32 = 42;              // Stack: instant
let y: Box<i32> = Box::new(42); // Heap: allocator call
let z: Vec<i32> = vec![1, 2, 3]; // Heap: allocator call
```

### Pre-allocating Collections

```rust
// Bad: grows and reallocates multiple times
let mut v = Vec::new();
for i in 0..10_000 {
    v.push(i); // may reallocate
}

// Good: single allocation
let mut v = Vec::with_capacity(10_000);
for i in 0..10_000 {
    v.push(i); // no reallocation
}
```

## Real World Context

High-frequency trading systems and game engines pre-allocate all memory at startup. Web servers use arena allocators for per-request memory to avoid fragmentation.

## Deep Dive

### `Cow` — Clone on Write

```rust
use std::borrow::Cow;

fn process(input: &str) -> Cow<'_, str> {
    if input.contains(' ') {
        Cow::Owned(input.replace(' ', "_")) // allocates
    } else {
        Cow::Borrowed(input) // no allocation
    }
}
```

### Small String Optimization

The `compact_str` or `smol_str` crates store short strings inline without heap allocation:

```rust
use compact_str::CompactString;

let s = CompactString::new("hello"); // No heap alloc (< 24 bytes)
```

### Arena Allocation

Arena allocators (e.g., `bumpalo`) allocate from a contiguous block and free everything at once:

```rust
use bumpalo::Bump;

let arena = Bump::new();
let x = arena.alloc(42);         // Fast bump allocation
let s = arena.alloc_str("hello"); // No individual free
// Everything freed when arena drops
```

### Reusing Allocations

```rust
let mut buffer = String::new();
for line in lines {
    buffer.clear(); // Keeps allocation, resets length
    buffer.push_str(line);
    process(&buffer);
}
```

## Common Pitfalls

- Calling `.to_string()` or `.clone()` in hot loops — use references or Cow instead.
- Collecting into a Vec just to iterate it — use iterator chains instead.
- Forgetting `with_capacity` when the size is known or estimable.

## Best Practices

- Use `Vec::with_capacity` and `String::with_capacity` when size is known.
- Reuse buffers with `.clear()` instead of creating new allocations.
- Use `Cow<str>` when a function sometimes needs to allocate and sometimes does not.
- Consider arena allocation for short-lived, batch-processed data.
- Profile with DHAT to find the biggest allocation sources.

## Summary

Minimize heap allocations through pre-allocation, buffer reuse, Cow, and arena allocators. Profile with DHAT to identify allocation-heavy code paths. Every avoided allocation is a performance win.

## Code Examples

**Cow and buffer reuse to minimize allocations**

```rust
use std::borrow::Cow;

// Cow avoids allocation when not needed
fn normalize_path(path: &str) -> Cow<'_, str> {
    if path.contains('\\') {
        Cow::Owned(path.replace('\\', "/"))
    } else {
        Cow::Borrowed(path)
    }
}

// Reusing a buffer in a loop
fn process_lines(lines: &[&str]) {
    let mut buf = String::with_capacity(256);
    for line in lines {
        buf.clear(); // reuse allocation
        buf.push_str("PREFIX: ");
        buf.push_str(line);
        println!("{buf}");
    }
}
```


## Resources

- [bumpalo — Arena Allocator](https://github.com/fitzgen/bumpalo) — Fast bump allocation arena for Rust
- [DHAT Heap Profiler](https://docs.rs/dhat/latest/dhat/) — Heap profiling for Rust programs

---

> 📘 *This lesson is part of the [Rust Performance](https://stanza.dev/courses/rust-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*