---
source_course: "rust-traits"
source_lesson: "rust-traits-const-expressions"
---

# Const Expressions

Use const generics in expressions:

```rust
fn double_size<const N: usize>() -> [i32; N * 2] {
    [0; N * 2]
}

let arr: [i32; 10] = double_size::<5>();
```

## Const Bounds (Workaround)

Rust doesn't have direct const bounds yet, but you can use this trick:

```rust
// Ensure N >= 1
struct NonEmptyArray<T, const N: usize>
where
    [(); N - 1]:,  // Fails compilation if N == 0
{
    data: [T; N],
}

// This works:
let arr = NonEmptyArray::<i32, 5> { data: [0; 5] };

// This fails at compile time:
// let arr = NonEmptyArray::<i32, 0> { data: [] };
// Error: overflow evaluating `0_usize - 1`
```

## Const Generic Trait Bounds

```rust
trait ArrayOps<const N: usize> {
    fn sum(&self) -> i32;
}

impl<const N: usize> ArrayOps<N> for [i32; N] {
    fn sum(&self) -> i32 {
        self.iter().sum()
    }
}

let arr = [1, 2, 3, 4, 5];
assert_eq!(arr.sum(), 15);
```

## Default Const Values (Nightly)

```rust
#![feature(generic_const_parameter_types)]

struct Buffer<T, const N: usize = 1024> {
    data: [T; N],
}

// Uses default size
let buf: Buffer<u8> = ...;

// Custom size
let buf: Buffer<u8, 4096> = ...;
```

## Code Examples

**Const generic buffer**

```rust
// Compile-time checked buffer operations
struct Buffer<const CAP: usize> {
    data: [u8; CAP],
    len: usize,
}

impl<const CAP: usize> Buffer<CAP> {
    const fn new() -> Self {
        Buffer {
            data: [0; CAP],
            len: 0,
        }
    }
    
    fn capacity(&self) -> usize {
        CAP  // Zero-cost: known at compile time
    }
    
    // Concatenate two buffers at compile time
    fn concat<const OTHER: usize>(
        self,
        other: Buffer<OTHER>,
    ) -> Buffer<{ CAP + OTHER }> {
        let mut result = Buffer::<{ CAP + OTHER }>::new();
        result.data[..CAP].copy_from_slice(&self.data);
        result.data[CAP..].copy_from_slice(&other.data);
        result.len = self.len + other.len;
        result
    }
}
```


---

> 📘 *This lesson is part of the [Advanced Traits & Generics](https://stanza.dev/courses/rust-traits) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*