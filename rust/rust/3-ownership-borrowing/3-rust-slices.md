---
source_course: "rust"
source_lesson: "rust-slices"
---

# The Slice Type

## Introduction

Slices let you reference a contiguous portion of a collection rather than the entire thing. They are one of Rust's most versatile tools for working with strings, arrays, and vectors without copying data. If you have worked with string views in C++ or array slices in Go, Rust slices fill the same role but with compile-time safety.

## Key Concepts

- **Slice**: A reference to a contiguous sequence of elements within a collection. It stores a pointer to the start and a length.
- **String slice (`&str`)**: A reference to a portion of a `String` (or a string literal). This is the most common slice type in Rust.
- **Array slice (`&[T]`)**: A reference to a portion of an array or vector.
- **Range syntax**: `[start..end]` creates a half-open range. `[..end]`, `[start..]`, and `[..]` are shorthands.

## Real World Context

Slices are used everywhere in production Rust code. Parsers use string slices to refer to portions of input without allocating new strings. Network protocols use byte slices (`&[u8]`) to inspect packet headers without copying memory. By accepting `&str` instead of `&String`, your functions become more flexible and idiomatic.

## Deep Dive

You create a string slice using range syntax on a `String`:

```rust
let s = String::from("hello world");
let hello = &s[0..5];   // "hello"
let world = &s[6..11];  // "world"
let whole = &s[..];     // "hello world"
```

A function that returns a string slice ties the returned reference to the input, so the compiler ensures the source data lives long enough:

```rust
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }
    &s[..]
}
```

This function works with both `String` references and string literals because it accepts `&str`.

Array slices work the same way:

```rust
let numbers = [1, 2, 3, 4, 5];
let middle: &[i32] = &numbers[1..4]; // [2, 3, 4]
```

Slices also prevent mutation bugs. If you hold a slice to a `String`, the compiler will not let you mutate the underlying `String`:

```rust
let mut s = String::from("hello world");
let word = first_word(&s);
// s.clear(); // Error! Cannot mutate while slice exists
println!("{}", word);
```

## Common Pitfalls

1. **Slicing on non-character boundaries in UTF-8 strings** — Rust strings are UTF-8, and slicing at a byte offset that falls in the middle of a multi-byte character will cause a panic at runtime. Always ensure your indices align with character boundaries or use `.char_indices()`.
2. **Confusing `&String` and `&str`** — Functions that accept `&String` cannot receive string literals directly. Prefer `&str` as the parameter type because it works with both `String` references and `&str` values.

## Best Practices

1. **Use `&str` in function signatures** — Accept `&str` instead of `&String` to make your functions compatible with literals, slices, and owned strings.
2. **Prefer iterators over manual indexing** — Instead of slicing strings by byte index, use `.chars()`, `.split_whitespace()`, or `.split()` to avoid UTF-8 boundary issues.

## Summary

- Slices are references to a contiguous portion of a collection, storing a pointer and a length.
- String slices (`&str`) are the most common; they reference part (or all) of a `String` or a string literal.
- Array slices (`&[T]`) let you work with subsections of arrays and vectors.
- Holding a slice prevents mutation of the underlying data, enforced by the borrow checker.
- Always prefer `&str` over `&String` in function parameters for maximum flexibility.

## Code Examples

**Flexible function signatures with &str**

```rust
fn main() {
    // Prefer &str over &String in function parameters
    fn print_length(s: &str) {
        println!("Length: {}", s.len());
    }
    
    let owned = String::from("hello");
    let literal = "world";
    
    print_length(&owned);  // Works with String
    print_length(literal); // Works with &str
    print_length(&owned[1..4]); // Works with slices
}
```


## Resources

- [The Slice Type](https://doc.rust-lang.org/stable/book/ch04-03-slices.html) — Official guide to slices in Rust
- [Rust By Example - Slices](https://doc.rust-lang.org/rust-by-example/primitives/array.html) — Practical examples of working with array slices

---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*