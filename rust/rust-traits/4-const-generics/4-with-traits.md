---
source_course: "rust-traits"
source_lesson: "rust-traits-const-generics-with-traits"
---

# Const Generics with Traits

## Introduction
Const generics become especially powerful when combined with traits. You can implement traits for types parameterized by const values, create trait bounds that depend on const parameters, and use const generics in associated types.

## Key Concepts
- **Const-Parameterized Trait Impl**: Implementing a trait for `[T; N]` for all values of `N`.
- **Const in Trait Bounds**: Using const parameters in where clauses and trait bound positions.
- **Const Generic Associated Types**: Traits whose associated types use const parameters.

## Real World Context
The standard library implements `Default`, `Debug`, `Clone`, `PartialEq`, and many other traits for `[T; N]` using const generics. Serialization libraries implement traits for arrays of any size. Embedded HALs use const generics to parameterize peripheral register sizes.

## Deep Dive

### Implementing Traits for All Array Sizes

```rust
use std::fmt;

struct FixedArray<T, const N: usize> {
    data: [T; N],
}

impl<T: fmt::Debug, const N: usize> fmt::Debug for FixedArray<T, N> {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        f.debug_struct("FixedArray")
            .field("len", &N)
            .field("data", &self.data.as_slice())
            .finish()
    }
}

impl<T: Default + Copy, const N: usize> Default for FixedArray<T, N> {
    fn default() -> Self {
        FixedArray { data: [T::default(); N] }
    }
}
```

### Trait Bounds on Const Parameters

You can write functions that require specific const generic relationships:

```rust
trait FixedSize {
    const SIZE: usize;
}

impl<T, const N: usize> FixedSize for [T; N] {
    const SIZE: usize = N;
}

fn print_size<T: FixedSize>() {
    println!("Size: {}", T::SIZE);
}
```

### Converting Between Sizes

```rust
struct SmallVec<T, const N: usize> {
    data: [Option<T>; N],
    len: usize,
}

impl<T: Copy + Default, const N: usize> SmallVec<T, N> {
    fn new() -> Self {
        SmallVec {
            data: [None; N],
            len: 0,
        }
    }

    fn as_slice(&self) -> &[Option<T>] {
        &self.data[..self.len]
    }
}

// Trait impl works for any size
impl<T: Copy + Default + PartialEq, const N: usize> PartialEq for SmallVec<T, N> {
    fn eq(&self, other: &Self) -> bool {
        self.len == other.len && self.as_slice() == other.as_slice()
    }
}
```

## Common Pitfalls
1. **Expecting different `N` values to be compatible** — `FixedArray<i32, 3>` and `FixedArray<i32, 5>` are different types. You cannot assign one to the other.
2. **Forgetting to propagate const generic params** — When implementing a trait for a const-generic type, the impl must also declare the const parameter.

## Best Practices
1. **Implement standard traits for const-generic types** — Always implement `Debug`, `Clone`, `PartialEq` where possible so your types work well in generic contexts.
2. **Use const generics in trait impls to avoid macros** — A single `impl<const N: usize>` replaces dozens of macro-generated impls.

## Summary
- Const generics can be used in trait implementations to cover all values of a const parameter.
- Traits can have associated constants derived from const generic parameters.
- Standard trait implementations (`Debug`, `Clone`, `PartialEq`) should be provided for const-generic types.
- Const generics in trait impls eliminate the need for macro-generated per-size impls.

## Code Examples

**A single trait implementation using const generics covers arrays of every size, replacing what previously required macro-generated impls for each size**

```rust
// Implementing a trait for all array sizes using const generics
trait Sum {
    fn sum(&self) -> i32;
}

impl<const N: usize> Sum for [i32; N] {
    fn sum(&self) -> i32 {
        self.iter().copied().sum()
    }
}

let a = [1, 2, 3];
let b = [10, 20, 30, 40, 50];
assert_eq!(a.sum(), 6);
assert_eq!(b.sum(), 150);
// One impl covers arrays of ANY size
```


## Resources

- [Const Generics](https://doc.rust-lang.org/reference/items/generics.html#const-generics) — Rust Reference on const generics and their use with traits

---

> 📘 *This lesson is part of the [Advanced Traits & Generics](https://stanza.dev/courses/rust-traits) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*