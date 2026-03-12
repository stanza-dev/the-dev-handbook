---
source_course: "rust-ownership"
source_lesson: "rust-ownership-cell-basics"
---

# Cell<T>: Copy-Based Interior Mutability

## Introduction
Rust's borrowing rules normally prevent mutation through shared references. Cell<T> provides a way to bypass this restriction for Copy types by never giving out references to the inner value — instead, it copies values in and out.

## Key Concepts
- **Interior mutability**: The ability to mutate data through a shared (`&`) reference, bypassing the usual `&mut` requirement.
- **Cell<T>**: A container that allows mutation of its inner value through `&self` methods, but only for types that implement `Copy`.
- **No reference to inner**: Cell never lets you borrow its contents — only copy them out with `get()` or replace them with `set()`.

## Real World Context
Counters embedded in shared structs, cached computed values, and configuration flags that need to change without requiring `&mut self` all benefit from Cell. It is common in GUI frameworks and event-driven code where callbacks hold shared references.

## Deep Dive
Cell allows mutation through a shared reference by working with copies:

```rust
use std::cell::Cell;

let x = Cell::new(5);
let y = &x;
let z = &x;

y.set(10);  // Mutate through shared ref!
z.set(20);

println!("{}", x.get());  // 20
```

The key insight is that Cell never gives you a reference to the inner value. The `get()` method returns a copy, and `set()` replaces the value entirely. This means there can never be a dangling reference to the old value.

A practical use case is a counter in a struct that does not require mutable access:

```rust
struct Stats {
    hits: Cell<usize>,
}

impl Stats {
    fn record_hit(&self) {  // Note: &self, not &mut self
        self.hits.set(self.hits.get() + 1);
    }
}
```

Cell is limited: it only works for `Copy` types, and it is not thread-safe (`!Sync`).

### LazyCell: Deferred Initialization

Since Rust 1.80, `LazyCell` provides single-threaded lazy initialization. The value is computed on first access and cached:

```rust
use std::cell::LazyCell;

let lazy = LazyCell::new(|| {
    println!("Computing...");
    42
});

println!("Before access");  // No computation yet
println!("Value: {}", *lazy); // Prints "Computing..." then "Value: 42"
println!("Value: {}", *lazy); // Just prints "Value: 42" (cached)
```

LazyCell is ideal for expensive computations that might not always be needed. Like Cell, it is `!Sync` and cannot be shared between threads — use `LazyLock` for that.

## Common Pitfalls
1. **Trying to use Cell with non-Copy types** — Cell requires the inner type to implement Copy. For String, Vec, or other owned types, use RefCell instead.
2. **Assuming Cell is thread-safe** — Cell is `!Sync`, meaning it cannot be shared between threads. Use atomic types or Mutex for thread-safe interior mutability.

## Best Practices
1. **Use Cell for small Copy values** — Counters, flags, and cached scalars are ideal Cell use cases.
2. **Prefer Cell over RefCell when possible** — Cell has zero runtime overhead (no borrow checking), making it the lighter option for Copy types.
3. **Use LazyCell for deferred single-threaded initialization** — When a value is expensive to compute and may not be needed, LazyCell avoids upfront cost.

## Summary
- Cell<T> provides interior mutability for Copy types.
- It works by copying values in and out, never giving out references.
- It has zero runtime overhead compared to RefCell's runtime borrow checking.
- LazyCell (Rust 1.80+) provides lazy initialization for single-threaded contexts.
- Cell is not thread-safe — use atomics or Mutex for concurrent access.

## Code Examples

**Counter with Cell — increment takes &self because Cell allows mutation through shared references**

```rust
use std::cell::Cell;

// Interior mutability in immutable struct
struct Counter {
    count: Cell<usize>,
}

impl Counter {
    fn new() -> Self {
        Counter { count: Cell::new(0) }
    }
    
    fn increment(&self) {  // Note: &self, not &mut self
        self.count.set(self.count.get() + 1);
    }
    
    fn get(&self) -> usize {
        self.count.get()
    }
}

let counter = Counter::new();
counter.increment();
counter.increment();
assert_eq!(counter.get(), 2);
```


## Resources

- [Cell Documentation](https://doc.rust-lang.org/std/cell/struct.Cell.html) — Official API reference for Cell

---

> 📘 *This lesson is part of the [Rust Ownership & Memory Model](https://stanza.dev/courses/rust-ownership) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*