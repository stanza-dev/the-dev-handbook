---
source_course: "rust"
source_lesson: "rust-ownership-rules"
---

# The 3 Rules of Ownership

1. Each value in Rust has an **owner**.
2. There can only be **one owner** at a time.
3. When the owner goes **out of scope**, the value is dropped (freed).

## The Move Semantics

When you assign a variable to another, ownership is transferred (moved). The old variable becomes invalid.

```rust
let s1 = String::from("hello");
let s2 = s1; // s1 is MOVED to s2
// println!("{}", s1); // Error! s1 is invalid
println!("{}", s2); // Works
```

## Why Move?

Rust avoids double-free bugs:

```
s1 ──────┐
         ├──► Heap: "hello"
s2 ──────┘
```

If both `s1` and `s2` were valid, dropping both would free the same memory twice!

## Copy vs Move

Types that implement the `Copy` trait are copied instead of moved:

```rust
let x = 5;
let y = x; // Copy, not move - both valid!
println!("x = {}, y = {}", x, y);
```

**Copy types:**
- All integer types (`i32`, `u64`, etc.)
- Boolean (`bool`)
- Floating point (`f32`, `f64`)
- Character (`char`)
- Tuples (if all elements are Copy)
- Arrays (if element type is Copy)

**NOT Copy (they Move):**
- `String`
- `Vec<T>`
- `Box<T>`
- Any type with heap allocation

See [What is Ownership?](https://doc.rust-lang.org/stable/book/ch04-01-what-is-ownership.html)

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