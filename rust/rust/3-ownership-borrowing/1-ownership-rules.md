---
source_course: "rust"
source_lesson: "rust-ownership-rules"
---

# The Three Rules of Ownership

## Introduction

Ownership is the cornerstone of Rust's memory safety guarantees. Unlike languages that use garbage collection or manual memory management, Rust enforces strict ownership rules at compile time. Understanding these rules is essential before writing any non-trivial Rust program.

## Key Concepts

- **Owner**: The variable that is responsible for a value and its memory. Each value has exactly one owner at a time.
- **Move**: When ownership transfers from one variable to another, the original variable becomes invalid.
- **Drop**: When the owner goes out of scope, Rust automatically frees the associated memory by calling the `drop` function.
- **Copy trait**: Certain simple types (integers, booleans, floats, chars) are duplicated instead of moved because they live entirely on the stack.

## Real World Context

In production systems, ownership prevents entire categories of memory bugs. Without ownership, you risk double-free errors, use-after-free vulnerabilities, and memory leaks. Languages like C and C++ leave these problems for the developer to manage manually. Rust's ownership model catches them all at compile time, meaning your deployed code is guaranteed free of these defects.

## Deep Dive

The three rules of ownership are:

1. Each value in Rust has an **owner**.
2. There can only be **one owner** at a time.
3. When the owner goes **out of scope**, the value is **dropped** (freed).

When you assign a heap-allocated value like a `String` to another variable, ownership moves:

```rust
let s1 = String::from("hello");
let s2 = s1; // s1 is MOVED to s2
// println!("{}", s1); // Error! s1 is invalid
println!("{}", s2); // Works fine
```

After the move, `s1` is no longer valid. This prevents both variables from trying to free the same memory.

For types that implement `Copy`, assignment duplicates the value instead:

```rust
let x = 5;
let y = x; // Copy, not move — both remain valid
println!("x = {}, y = {}", x, y);
```

Passing values to functions also transfers ownership:

```rust
fn takes_ownership(s: String) {
    println!("{}", s);
} // s is dropped here

let s = String::from("hello");
takes_ownership(s);
// s is no longer valid here
```

## Common Pitfalls

1. **Using a variable after it has been moved** — This is the most common ownership error. After assigning a `String` or `Vec` to another variable or passing it to a function, the original is invalid. Use references (`&`) or `.clone()` when you need to keep both.
2. **Assuming all types are moved** — Simple scalar types like `i32`, `bool`, and `f64` implement `Copy` and are duplicated automatically. Only heap-allocated types like `String`, `Vec<T>`, and `Box<T>` are moved.

## Best Practices

1. **Prefer borrowing over transferring ownership** — Pass references (`&T`) to functions whenever possible so the caller retains ownership of its data.
2. **Use `.clone()` deliberately** — Cloning is explicit in Rust. Only clone when you truly need an independent copy, and be aware of the performance cost for large data.

## Summary

- Every value has exactly one owner; when the owner goes out of scope, the value is dropped.
- Assigning heap-allocated types transfers ownership (move semantics); the original variable becomes invalid.
- Stack-only types that implement `Copy` are duplicated instead of moved.
- Passing a value to a function moves it unless you pass a reference.
- The compiler enforces all ownership rules at compile time with zero runtime cost.

## Code Examples

**Ownership and function calls**

```rust
fn main() {
    let s = String::from("hello");
    takes_ownership(s);
    // println!("{}", s); // Error! s was moved
    
    let x = 5;
    makes_copy(x);
    println!("{}", x); // Works! x was copied
}

fn takes_ownership(some_string: String) {
    println!("{}", some_string);
} // some_string is dropped here

fn makes_copy(some_integer: i32) {
    println!("{}", some_integer);
} // some_integer goes out of scope, nothing special
```


## Resources

- [Understanding Ownership](https://doc.rust-lang.org/stable/book/ch04-01-what-is-ownership.html) — The official guide to ownership

---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*