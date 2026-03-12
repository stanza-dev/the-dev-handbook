---
source_course: "rust"
source_lesson: "rust-vectors"
---

# Vectors: Dynamic Arrays

## Introduction

Rust arrays have a fixed size known at compile time, but real programs often need collections that grow and shrink. The `Vec<T>` type is Rust's dynamically-sized array, stored on the heap, and it is the most commonly used collection in the language.

## Key Concepts

- **Vec<T>**: A growable, heap-allocated list of elements of type `T`. It owns its data and frees memory when dropped.
- **Capacity vs Length**: Length is how many elements the vector holds. Capacity is how much memory is allocated. When length exceeds capacity, the vector reallocates.
- **Safe access**: Using `.get()` returns `Option<&T>` instead of panicking on out-of-bounds access.

## Real World Context

Vectors are everywhere in production Rust: collecting results from a database query, buffering network packets, storing parsed tokens in a compiler. Any time you need a list whose size is not known at compile time, you reach for `Vec<T>`.

## Deep Dive

You can create a vector in two ways. The `Vec::new()` constructor creates an empty vector, while the `vec![]` macro provides shorthand with initial values:

```rust
let mut v: Vec<i32> = Vec::new();
v.push(1);
v.push(2);

let v2 = vec![10, 20, 30];
```

Accessing elements can be done with indexing or `.get()`. Indexing with `&v[i]` panics if `i` is out of bounds, whereas `.get(i)` returns `Option<&T>`:

```rust
let v = vec![1, 2, 3];
let third = &v[2];           // panics if out of bounds
let maybe = v.get(10);       // returns None safely
```

The `.get()` approach is preferred when the index comes from user input or external data.

Iterating over a vector borrows its elements. You can iterate immutably or mutably:

```rust
let mut v = vec![100, 32, 57];
for val in &mut v {
    *val += 50; // dereference to modify
}
```

The dereference operator `*` is needed because `val` is `&mut i32`.

Useful methods include `push`, `pop`, `insert`, `remove`, `len`, `is_empty`, `contains`, `sort`, and `dedup`. The `pop` method returns `Option<T>`, returning `None` if the vector is empty.

## Common Pitfalls

1. **Borrowing while mutating** — You cannot hold an immutable reference to a vector element and then push to the vector. The push may reallocate, invalidating the reference. The borrow checker prevents this at compile time.
2. **Indexing with unchecked bounds** — Using `v[i]` with an index from user input can cause a panic. Prefer `v.get(i)` and handle the `None` case.

## Best Practices

1. **Pre-allocate with `Vec::with_capacity(n)`** — If you know approximately how many elements you will store, pre-allocating avoids repeated reallocations and improves performance.
2. **Use iterators over indexing** — Idiomatic Rust favors `for item in &v` over `for i in 0..v.len()`. Iterator-based code is safer, often faster, and more readable.

## Summary

- `Vec<T>` is Rust's growable, heap-allocated array.
- Use `vec![]` for quick initialization and `Vec::with_capacity` for performance-sensitive code.
- Prefer `.get()` over indexing when the index may be out of bounds.
- The borrow checker prevents simultaneous mutation and borrowing.
- Iterate with `&v` (immutable) or `&mut v` (mutable) references.

## Code Examples

**Heterogeneous vectors via enum**

```rust
// Store different types with enum
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Text(String::from("blue")),
    SpreadsheetCell::Float(10.12),
];
```


## Resources

- [Storing Lists of Values with Vectors](https://doc.rust-lang.org/stable/book/ch08-01-vectors.html) — Official Rust Book chapter on Vec<T> with examples and ownership rules
- [Vec API Reference](https://doc.rust-lang.org/std/vec/struct.Vec.html) — Complete standard library documentation for Vec<T>

---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*