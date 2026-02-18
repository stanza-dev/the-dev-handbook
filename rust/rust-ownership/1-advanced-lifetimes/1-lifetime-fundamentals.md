---
source_course: "rust-ownership"
source_lesson: "rust-ownership-lifetime-fundamentals"
---

# The Purpose of Lifetimes

Lifetimes are Rust's way of ensuring references are always valid.

## The Dangling Reference Problem

```rust
fn dangling() -> &String {
    let s = String::from("hello");
    &s  // Error! s is dropped when function returns
}     // Reference would point to freed memory
```

## Lifetime = Scope of Validity

Every reference has a lifetime - the region of code where it's valid:

```rust
fn main() {
    let r;                  // ---------+-- 'a
                            //          |
    {                       //          |
        let x = 5;          // -+-- 'b  |
        r = &x;             //  |       |
    }                       // -+       |
                            //          |
    println!("{}", r);      // Error! 'b ended before use
}                           // ---------+
```

## The Borrow Checker

Rust's borrow checker compares lifetimes at compile time:

```rust
fn main() {
    let x = 5;            // ----------+-- 'a
    let r = &x;           // --+-- 'b  |
                          //   |       |
    println!("{}", r);    //   |       |
}                         // --+-------+
// 'b fits entirely within 'a ✓
```

## When You Don't See Lifetimes

Most lifetimes are inferred. You only annotate when the compiler can't figure it out.

See [Validating References with Lifetimes](https://doc.rust-lang.org/stable/book/ch10-03-lifetime-syntax.html).

## Code Examples

**Lifetimes describe relationships**

```rust
// Lifetime annotations don't change how long values live!
// They just describe relationships for the compiler.

// This tells the compiler: the return value lives as long
// as the shorter of 'a and 'b
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
```


## Resources

- [Lifetimes](https://doc.rust-lang.org/stable/book/ch10-03-lifetime-syntax.html) — Official guide to lifetimes

---

> 📘 *This lesson is part of the [Rust Ownership & Memory Model](https://stanza.dev/courses/rust-ownership) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*