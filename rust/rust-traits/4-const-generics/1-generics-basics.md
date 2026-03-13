---
source_course: "rust-traits"
source_lesson: "rust-traits-const-generics-basics"
---

# Const Generics Basics

## Introduction
Const generics (stable since Rust 1.51) let you parameterize types and functions by compile-time constant values. Since Rust 1.89, you can also use `_` for const generic arguments to let the compiler infer the value.

## Key Concepts
- **Const Generic Parameter**: A compile-time constant declared with `const N: usize` in a generic parameter list.
- **Const Inference**: Using `_` as a const generic argument (Rust 1.89+) to let the compiler infer the value from context.
- **Allowed Types**: Stable const generic parameters support integers (`usize`, `i32`, etc.), `bool`, and `char`.

## Real World Context
Before const generics, the standard library had to implement traits for arrays up to size 32 using macros. With const generics, `impl<T, const N: usize> Trait for [T; N]` covers all sizes in one line.

## Deep Dive

### Basic Usage

Parameterize types by compile-time values:

```rust
struct Array<T, const N: usize> {
    data: [T; N],
}

impl<T, const N: usize> Array<T, N> {
    fn len(&self) -> usize { N } // Known at compile time
}

let arr: Array<i32, 5> = Array { data: [0; 5] };
assert_eq!(arr.len(), 5);
```

### Solving the Array Problem

```rust
// Before: manual impls for each size
impl<T> MyTrait for [T; 0] { }
impl<T> MyTrait for [T; 1] { }
// ... tedious!

// After: one impl for all sizes
impl<T, const N: usize> MyTrait for [T; N] { }
```

### Const Generic Inference (Rust 1.89+)

Use `_` to let the compiler infer the const value:

```rust
fn create_array<const N: usize>() -> [i32; N] { [0; N] }

let arr: [i32; 3] = create_array::<_>(); // Compiler infers N = 3
let arr: [i32; 5] = create_array();       // Also inferred from context
```

### Allowed Types

Stable const generic parameter types:
- Integers: `usize`, `i8`, `i16`, `i32`, `i64`, `i128`, `u8`, `u16`, `u32`, `u64`, `u128`, `isize`
- `bool`
- `char`

Not yet stable for const generics: floats, `&str`, enums, and custom types.

## Common Pitfalls
1. **Using floats as const generics** — Floats are not yet stable for const generic parameters. Use integers or newtype wrappers.
2. **Complex const expressions on stable** — Not all const expressions compile on stable Rust. Generic const expressions involving multiple parameters (e.g., `{ N + M }`) may require careful handling.

## Best Practices
1. **Use const generics for array and buffer sizes** — They eliminate the need for macro-generated impls and make APIs size-generic.
2. **Let the compiler infer with `_`** — Since Rust 1.89, prefer inference over explicit turbofish when the context makes the value clear.

## Summary
- Const generics parameterize types by compile-time constants.
- Supported types: integers, `bool`, `char`.
- Rust 1.89 added `_` inference for const generic arguments.
- They eliminate the need for per-size trait implementations.

## Code Examples

**A compile-time-checked matrix type using const generics for row and column dimensions, with a transpose method that swaps the dimensions in the type**

```rust
// Matrix with const dimensions
struct Matrix<T, const ROWS: usize, const COLS: usize> {
    data: [[T; COLS]; ROWS],
}

impl<T: Default + Copy, const R: usize, const C: usize> Matrix<T, R, C> {
    fn new() -> Self {
        Matrix { data: [[T::default(); C]; R] }
    }

    fn transpose(self) -> Matrix<T, C, R> {
        let mut result = Matrix::<T, C, R>::new();
        for i in 0..R {
            for j in 0..C {
                result.data[j][i] = self.data[i][j];
            }
        }
        result
    }
}

let m: Matrix<i32, 2, 3> = Matrix::new();
let t: Matrix<i32, 3, 2> = m.transpose();
// Dimensions are checked at compile time!
```


## Resources

- [Const Generics](https://doc.rust-lang.org/reference/items/generics.html#const-generics) — Rust Reference section on const generic parameters

---

> 📘 *This lesson is part of the [Advanced Traits & Generics](https://stanza.dev/courses/rust-traits) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*