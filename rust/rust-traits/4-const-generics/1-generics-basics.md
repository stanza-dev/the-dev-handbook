---
source_course: "rust-traits"
source_lesson: "rust-traits-const-generics-basics"
---

# Const Generics (Stable Since 1.51)

Parameterize types by compile-time constant values:

```rust
// Array with generic size
struct Array<T, const N: usize> {
    data: [T; N],
}

impl<T, const N: usize> Array<T, N> {
    fn len(&self) -> usize {
        N  // Known at compile time!
    }
}

let arr: Array<i32, 5> = Array { data: [0; 5] };
assert_eq!(arr.len(), 5);
```

## Solving the Array Problem

Before const generics:

```rust
// Had to implement for each size separately!
impl<T> MyTrait for [T; 0] { }
impl<T> MyTrait for [T; 1] { }
impl<T> MyTrait for [T; 2] { }
// ... up to 32
```

With const generics:

```rust
impl<T, const N: usize> MyTrait for [T; N] { }
// Works for ALL sizes!
```

## Functions with Const Generics

```rust
fn create_array<const N: usize>() -> [i32; N] {
    [0; N]
}

let a: [i32; 3] = create_array();
let b: [i32; 100] = create_array();
```

## Allowed Types for const Parameters

- Integers: `usize`, `i32`, etc.
- `bool`
- `char`

Not yet stable: floats, &str, custom types.

See [Const Generics](https://doc.rust-lang.org/reference/items/generics.html#const-generics).

## Code Examples

**Const generic matrix**

```rust
// Matrix with const dimensions
struct Matrix<T, const ROWS: usize, const COLS: usize> {
    data: [[T; COLS]; ROWS],
}

impl<T: Default + Copy, const R: usize, const C: usize> Matrix<T, R, C> {
    fn new() -> Self {
        Matrix {
            data: [[T::default(); C]; R],
        }
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
```


---

> 📘 *This lesson is part of the [Advanced Traits & Generics](https://stanza.dev/courses/rust-traits) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*