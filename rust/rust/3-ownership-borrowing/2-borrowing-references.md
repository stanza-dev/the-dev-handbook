---
source_course: "rust"
source_lesson: "rust-borrowing-references"
---

# Borrowing & References

## Introduction

Moving ownership every time you call a function would be incredibly tedious. Borrowing lets you give a function temporary access to a value without transferring ownership. This is one of Rust's most frequently used mechanisms and is essential for writing ergonomic, safe code.

## Key Concepts

- **Reference (`&T`)**: An immutable borrow that lets you read a value without owning it. The original owner retains ownership.
- **Mutable reference (`&mut T`)**: A borrow that lets you both read and modify the value. Only one mutable reference can exist at a time.
- **Borrow checker**: The compiler component that enforces borrowing rules at compile time, preventing data races and dangling references.

## Real World Context

Every Rust function that reads data without consuming it uses references. In a web server, for example, request handlers borrow the shared application state rather than cloning it for each request. The borrow checker guarantees that no two handlers can mutate the same state simultaneously, eliminating an entire class of concurrency bugs.

## Deep Dive

You create a reference with the `&` operator. The function receives access to the data without taking ownership:

```rust
fn calculate_length(s: &String) -> usize {
    s.len()
}

let s1 = String::from("hello");
let len = calculate_length(&s1);
println!("'{}' has length {}.", s1, len); // s1 is still valid
```

The borrow checker enforces two rules. At any given time, you can have either **one mutable reference** or **any number of immutable references**, but never both simultaneously:

```rust
let mut s = String::from("hello");
let r1 = &s;     // OK — immutable borrow
let r2 = &s;     // OK — multiple immutable borrows allowed
println!("{} and {}", r1, r2);
// r1 and r2 are no longer used after this point
let r3 = &mut s; // OK — no active immutable borrows
r3.push_str(", world");
```

Mutable references let you modify borrowed data:

```rust
fn append_greeting(s: &mut String) {
    s.push_str(", world");
}

let mut s = String::from("hello");
append_greeting(&mut s);
println!("{}", s); // "hello, world"
```

## Common Pitfalls

1. **Mixing mutable and immutable borrows** — You cannot hold an immutable reference and a mutable reference to the same data at the same time. Move the immutable usage above the mutable borrow so the compiler sees they do not overlap.
2. **Forgetting `mut` on the variable** — To create a `&mut` reference, the underlying variable must be declared with `let mut`. A common error is trying to mutably borrow an immutable binding.

## Best Practices

1. **Accept `&str` instead of `&String` in function parameters** — `&str` is more flexible because it accepts both `String` references and string literals without extra conversion.
2. **Keep borrow scopes as short as possible** — The shorter the lifetime of a reference, the less likely you are to run into borrow checker conflicts.

## Summary

- References (`&T`) let you borrow data without taking ownership.
- Mutable references (`&mut T`) allow modification but enforce exclusive access.
- You may have many immutable references or one mutable reference, never both at once.
- The borrow checker enforces these rules at compile time, preventing data races.
- Prefer `&str` over `&String` for function parameters to maximize flexibility.

## Code Examples

**Preventing iterator invalidation**

```rust
fn main() {
    let mut v = vec![1, 2, 3];
    
    // This would be a data race in C++!
    // let first = &v[0];  // Immutable borrow
    // v.push(4);          // Mutable borrow - Error!
    // println!("{}", first);
    
    // Correct approach:
    v.push(4);
    let first = &v[0];
    println!("First: {}", first);
}
```


## Resources

- [References and Borrowing](https://doc.rust-lang.org/stable/book/ch04-02-references-and-borrowing.html) — Official guide to references and borrowing in Rust
- [The Rustonomicon - References](https://doc.rust-lang.org/nomicon/references.html) — Advanced details on how references work under the hood

---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*