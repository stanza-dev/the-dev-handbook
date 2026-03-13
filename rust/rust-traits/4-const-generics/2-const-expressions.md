---
source_course: "rust-traits"
source_lesson: "rust-traits-const-expressions"
---

# Const Expressions & Constraints

## Introduction
Const generics can be used in trait bounds and default values on stable Rust. Arithmetic on const generic parameters (like `{ N * 2 }` or `{ A + B }`) requires the nightly-only `generic_const_exprs` feature. This lesson covers both stable and nightly patterns.

## Key Concepts
- **Const Expression**: Using const generics in arithmetic expressions like `{ N * 2 }` or `{ A + B }`.
- **Const Generic Trait Bound**: Using a const parameter in a trait implementation: `impl<const N: usize> Trait for [T; N]`.
- **Compile-Time Validation**: Using const expressions to enforce invariants at compile time.

## Real World Context
Networking libraries use const generics to define packet buffers whose sizes are computed at compile time. Cryptography crates use them to ensure key sizes match algorithm requirements.

## Deep Dive

### Const Generic Trait Bounds (Stable)

Using const parameters in trait implementations is stable:

```rust
trait ArrayOps<const N: usize> {
    fn sum(&self) -> i32;
}

impl<const N: usize> ArrayOps<N> for [i32; N] {
    fn sum(&self) -> i32 { self.iter().sum() }
}

let arr = [1, 2, 3, 4, 5];
assert_eq!(arr.sum(), 15);
```

### Default Values for Const Parameters

Default const values have been stable since Rust 1.59:

```rust
// Default const generic value (stable since Rust 1.59)
struct Buffer<T, const N: usize = 1024> {
    data: [T; N],
}

let buf = Buffer::<u8>::new();       // N defaults to 1024
let small = Buffer::<u8, 64>::new(); // Override to 64
```

### Nightly: Const Generic Expressions

**This requires nightly Rust.** Arithmetic on const generic parameters (like `N * 2` or `{ CAP + OTHER }`) requires the `generic_const_exprs` feature gate.

```rust
#![feature(generic_const_exprs)]

fn double_size<const N: usize>() -> [i32; N * 2] {
    [0; N * 2]
}

let arr: [i32; 10] = double_size::<5>();
```

**Nightly-only feature:** Buffer concatenation with computed const sizes:

```rust
#![feature(generic_const_exprs)]

struct Buffer<const CAP: usize> {
    data: [u8; CAP],
    len: usize,
}

impl<const CAP: usize> Buffer<CAP> {
    const fn new() -> Self {
        Buffer { data: [0; CAP], len: 0 }
    }

    fn concat<const OTHER: usize>(
        self, other: Buffer<OTHER>
    ) -> Buffer<{ CAP + OTHER }> {
        let mut result = Buffer::<{ CAP + OTHER }>::new();
        result.data[..CAP].copy_from_slice(&self.data);
        result.data[CAP..].copy_from_slice(&other.data);
        result.len = self.len + other.len;
        result
    }
}
```

The resulting buffer has capacity `CAP + OTHER`, computed and checked at compile time — but this pattern requires nightly Rust.

## Common Pitfalls
1. **Complex const expressions on stable** — Not all const expressions compile on stable Rust. Stick to simple arithmetic (`+`, `*`, `-`) in const generic contexts.
2. **Compile errors are cryptic** — Const expression evaluation failures produce hard-to-read error messages. Start simple and add complexity incrementally.

## Best Practices
1. **Wrap expressions in braces** — Use `{ N + M }` with braces when using const expressions in type positions.
2. **Use `const fn` for helper logic** — Extract complex const computations into `const fn` functions for clarity.

## Summary
- Const generic trait bounds (`impl<const N: usize> Trait for [T; N]`) are stable.
- Default const values (`const N: usize = 1024`) are stable since Rust 1.59.
- Const generic expressions (`[T; N * 2]`, `Buffer<{ A + B }>`) require nightly (`#![feature(generic_const_exprs)]`).
- Wrap const expressions in braces in type positions.

## Code Examples

**A buffer whose capacity is a const generic parameter — the capacity is known at compile time with zero runtime cost**

```rust
// Compile-time checked buffer with const generic capacity
struct Buffer<const CAP: usize> {
    data: [u8; CAP],
    len: usize,
}

impl<const CAP: usize> Buffer<CAP> {
    const fn new() -> Self {
        Buffer { data: [0; CAP], len: 0 }
    }

    fn capacity(&self) -> usize {
        CAP // Zero-cost: known at compile time
    }
}

let small = Buffer::<64>::new();
let large = Buffer::<4096>::new();
assert_eq!(small.capacity(), 64);
assert_eq!(large.capacity(), 4096);
```


## Resources

- [Const Generics RFC](https://doc.rust-lang.org/reference/items/generics.html#const-generics) — Rust Reference on const generics, including expression support

---

> 📘 *This lesson is part of the [Advanced Traits & Generics](https://stanza.dev/courses/rust-traits) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*