---
source_course: "rust"
source_lesson: "rust-scalar-types"
---

# Scalar Types

Scalar types represent a single value. Rust has four primary scalar types.

## Integer Types

| Length | Signed | Unsigned |
|--------|--------|----------|
| 8-bit  | `i8`   | `u8`     |
| 16-bit | `i16`  | `u16`    |
| 32-bit | `i32`  | `u32`    |
| 64-bit | `i64`  | `u64`    |
| 128-bit| `i128` | `u128`   |
| arch   | `isize`| `usize`  |

**Integer literals:**
```rust
let decimal = 98_222;     // Underscores for readability
let hex = 0xff;
let octal = 0o77;
let binary = 0b1111_0000;
let byte = b'A';          // u8 only
```

## Floating-Point Types

```rust
let x = 2.0;      // f64 (default)
let y: f32 = 3.0; // f32
```

**Important:** Floats can't be used for indexing or exact comparisons!

## Boolean Type

```rust
let t = true;
let f: bool = false;
```

## Character Type

Rust's `char` is 4 bytes - a Unicode Scalar Value:

```rust
let c = 'z';
let z: char = 'ℤ';
let heart_eyed_cat = '😻';
```

## Type Inference & Annotations

Rust infers types, but you can annotate:

```rust
let guess: u32 = "42".parse().expect("Not a number!");
// Without : u32, Rust doesn't know which number type to parse into
```

See [Data Types](https://doc.rust-lang.org/stable/book/ch03-02-data-types.html).

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


---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*