---
source_course: "rust"
source_lesson: "rust-scalar-types"
---

# Scalar Types: Integers, Floats, Booleans & Characters

## Introduction

Scalar types represent a single value, and they are the foundation of every Rust program. Rust provides four primary scalar types: integers, floating-point numbers, booleans, and characters. Unlike dynamically typed languages, Rust requires you to understand these types because the compiler uses them to catch bugs and optimize your code.

## Key Concepts

- **Integer types**: Whole numbers that come in signed (`i8` to `i128`) and unsigned (`u8` to `u128`) variants, plus architecture-dependent `isize` and `usize`.
- **Floating-point types**: Decimal numbers available as `f32` (single precision) and `f64` (double precision, the default).
- **Boolean type**: `bool` holds either `true` or `false` and occupies one byte.
- **Character type**: `char` represents a Unicode Scalar Value and is four bytes wide, supporting emoji and international scripts.
- **Type inference**: Rust can deduce types from context, but sometimes you must annotate explicitly (e.g., when parsing strings).

## Real World Context

Choosing the right integer type matters in systems programming. Using `u8` for pixel values saves memory when processing millions of pixels. Using `usize` for collection indices is required by the standard library. Understanding overflow behavior is critical for financial calculations and cryptographic code. In production Rust, incorrect type choices lead to panics or subtle wrapping bugs.

## Deep Dive

Rust provides integer types at various bit widths:

| Length | Signed | Unsigned |
|--------|--------|----------|
| 8-bit  | `i8`   | `u8`     |
| 16-bit | `i16`  | `u16`    |
| 32-bit | `i32`  | `u32`    |
| 64-bit | `i64`  | `u64`    |
| 128-bit| `i128` | `u128`   |
| arch   | `isize`| `usize`  |

The default integer type is `i32`, which is generally the fastest even on 64-bit systems. Integer literals support several formats for readability:

```rust
let decimal = 98_222;     // Underscores for readability
let hex = 0xff;
let octal = 0o77;
let binary = 0b1111_0000;
let byte = b'A';          // u8 only
```

Underscores in numeric literals are purely visual and have no effect on the value.

For floating-point numbers, `f64` is the default because it offers double precision with negligible speed difference on modern CPUs:

```rust
let x = 2.0;      // f64 (default)
let y: f32 = 3.0; // f32, explicit annotation required
```

The `char` type is four bytes and represents a Unicode Scalar Value, meaning it can hold far more than just ASCII:

```rust
let c = 'z';
let z: char = 'ℤ';
let heart_eyed_cat = '😻';
```

Type inference works in most cases, but you must annotate when the compiler cannot deduce the type:

```rust
let guess: u32 = "42".parse().expect("Not a number!");
// Without : u32, Rust doesn't know which number type to parse into
```

## Common Pitfalls

1. **Integer overflow in debug vs release** — In debug mode, integer overflow panics (crashes). In release mode, it silently wraps around (e.g., `255u8 + 1` becomes `0`). Use explicit methods like `checked_add`, `wrapping_add`, or `saturating_add` to handle overflow intentionally.
2. **Using floats for equality checks** — Floating-point arithmetic can produce tiny rounding errors. Never compare floats with `==`; use an epsilon threshold or the `approx` crate instead.
3. **Confusing `char` with `u8`** — A `char` is four bytes (Unicode), while a `u8` is one byte (ASCII). String bytes and characters are different things in Rust.

## Best Practices

1. **Use `i32` as your default integer** — Unless you have a specific reason (memory constraints, API requirements), `i32` is the fastest and most common choice.
2. **Use `usize` for indexing** — Collection indices and lengths are always `usize`. Trying to index with `i32` will not compile.
3. **Handle overflow explicitly** — Use `checked_add`, `saturating_add`, or `wrapping_add` instead of relying on debug-mode panics or release-mode wrapping.

## Summary

- Rust has four scalar types: integers, floats, booleans, and characters.
- The default integer is `i32`; the default float is `f64`.
- Integer overflow panics in debug mode but wraps in release mode — handle it explicitly.
- `char` is four bytes and supports full Unicode.
- Use type annotations when the compiler cannot infer the type from context.

## Code Examples

**Integer overflow handling**

```rust
fn main() {
    // Integer overflow in debug mode panics!
    let x: u8 = 255;
    // let y = x + 1; // Panics in debug, wraps in release
    
    // Safe overflow handling
    let wrapped = x.wrapping_add(1);  // 0
    let checked = x.checked_add(1);   // None
    let saturated = x.saturating_add(1); // 255
}
```


## Resources

- [Data Types](https://doc.rust-lang.org/stable/book/ch03-02-data-types.html) — Official Rust Book chapter covering scalar and compound types
- [Primitive Types - Rust Standard Library](https://doc.rust-lang.org/std/#primitives) — Standard library reference for all primitive types

---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*