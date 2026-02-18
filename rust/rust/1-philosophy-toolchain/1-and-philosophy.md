---
source_course: "rust"
source_lesson: "rust-why-and-philosophy"
---

# Why Rust?

Rust solves a problem that has plagued software for decades: **Memory Safety without Garbage Collection**.

## The Problem with Other Languages

**C/C++:** Fast but prone to memory bugs:
- Buffer overflows
- Use-after-free
- Data races
- Null pointer dereferences

**Java/Python/Go:** Safe but have runtime overhead:
- Garbage collection pauses
- Higher memory usage
- Less predictable performance

## Rust's Solution: Ownership

Rust introduces a **compile-time ownership system** that guarantees:

1. **Memory safety** - No dangling pointers or buffer overflows
2. **Thread safety** - Data races are impossible
3. **Zero-cost abstractions** - High-level code compiles to optimal machine code

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1; // s1 is MOVED to s2
    // println!("{}", s1); // Compile error! Rust prevents use-after-move
    println!("{}", s2); // Works perfectly
}
```

## Zero-Cost Abstractions

Rust's motto: **"What you don't use, you don't pay for."**

Iterator chains compile to the same assembly as hand-written loops:

```rust
// High-level, readable code
let sum: i32 = (1..1000).filter(|x| x % 2 == 0).sum();

// Compiles to optimal machine code - no iterator objects at runtime!
```

See [The Rust Programming Language](https://doc.rust-lang.org/stable/book/ch00-00-introduction.html).

## Code Examples

**Compile-time safety**

```rust
// Rust catches bugs at compile time
fn main() {
    let v = vec![1, 2, 3];
    let first = &v[0];
    // v.push(4); // Error! Can't mutate while borrowed
    println!("{}", first);
}
```


## Resources

- [The Rust Programming Language Book](https://doc.rust-lang.org/stable/book/) — The official Rust book - your primary learning resource

---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*