---
source_course: "rust-unsafe"
source_lesson: "rust-unsafe-transmute-layout"
---

# Transmute & Memory Layout

## Introduction
`std::mem::transmute` reinterprets the bits of one type as another type. It is one of the most powerful and dangerous operations in Rust. Understanding when it is valid — and when safer alternatives exist — will help you avoid subtle undefined behavior.

## Key Concepts
- **`transmute<T, U>`**: Reinterprets a value of type `T` as type `U`. Both types must have the same size.
- **`transmute_copy`**: Like transmute but works with references and does not consume the source.
- **`#[repr(C)]`**: Guarantees C-compatible struct layout (field order preserved, predictable padding).
- **`#[repr(transparent)]`**: Guarantees a single-field struct has the same layout as its field.

## Real World Context
Transmute is used in serialization libraries (interpreting bytes as structs), FFI (converting between equivalent C and Rust types), and networking code (reading packet headers). The standard library uses it internally for `NonZeroU32` and similar types.

## Deep Dive

### Basic Transmute

```rust
use std::mem::transmute;

// Reinterpret f32 bits as u32 (same size: 4 bytes)
let float_bits: u32 = unsafe { transmute(1.0f32) };
assert_eq!(float_bits, 0x3F800000);
```

Both types must be the same size; the compiler checks this at compile time.

### When Transmute is Valid

Transmute is safe when:
1. Both types have the same size.
2. Every bit pattern of the source type is a valid bit pattern of the target type.

```rust
// SAFE: both are 4 bytes, and every u32 bit pattern is valid as [u8; 4]
let bytes: [u8; 4] = unsafe { transmute(0xDEADBEEFu32) };

// SAFE: #[repr(transparent)] guarantees same layout
#[repr(transparent)]
struct Wrapper(u32);
let w: Wrapper = unsafe { transmute(42u32) };
```

### When Transmute is UB

```rust
// UB: 0xFF is not a valid bool (must be 0 or 1)
let b: bool = unsafe { transmute(0xFFu8) };

// UB: 0 is not a valid NonZeroU32
let nz: std::num::NonZeroU32 = unsafe { transmute(0u32) };

// UB: null is not a valid reference
let r: &i32 = unsafe { transmute(0usize) };
```

The compiler may exploit these validity requirements for optimization, so UB here can cause any behavior.

### Safer Alternatives

Often you do not need transmute:

```rust
// Instead of transmute for float/int conversion:
let bits = f32::to_bits(1.0f32);  // u32
let float = f32::from_bits(bits); // f32

// Instead of transmute for byte conversion:
let bytes = 42u32.to_ne_bytes();  // [u8; 4]
let value = u32::from_ne_bytes(bytes);

// Instead of transmute for pointer casts:
let ptr: *const u8 = some_ptr as *const u8; // simple cast
```

Prefer these dedicated methods — they are safer and more readable.

### Memory Layout Control

Use repr attributes to control layout:

```rust
#[repr(C)]          // C-compatible layout (predictable field order)
#[repr(packed)]     // No padding between fields (may cause unaligned access)
#[repr(align(64))]  // Minimum alignment of 64 bytes (cache line alignment)
struct CacheLine {
    data: [u8; 64],
}
```

Without `#[repr(C)]`, Rust may reorder fields for optimization.

## Common Pitfalls
1. **Transmuting to types with validity invariants** — `bool`, `char`, `NonZero*`, and references all have restricted bit patterns. Random bits are usually invalid.
2. **Using transmute when `as` suffices** — For numeric casts and pointer casts, use `as`. Transmute is for reinterpreting bits, not converting values.
3. **Forgetting about padding** — Struct padding bytes are uninitialized. Transmuting a struct with padding to a byte array reads uninitialized bytes (UB).

## Best Practices
1. **Prefer `to_bits`/`from_bits` for float-integer conversion** — Safer and more readable than transmute.
2. **Use `#[repr(C)]` or `#[repr(transparent)]` before transmuting structs** — Without these, Rust's layout is not guaranteed.
3. **Use `bytemuck` for safe transmutation** — The `bytemuck` crate provides checked transmute for plain-old-data types.

## Summary
- `transmute` reinterprets bits from one type to another of the same size.
- It is UB if the target type has validity invariants that the source bits violate.
- Prefer safer alternatives: `to_bits`, `to_ne_bytes`, `as` casts, and `bytemuck`.
- Use `#[repr(C)]` or `#[repr(transparent)]` when transmuting structs.
- Only use transmute when no safer alternative exists.

## Code Examples

**Safer alternatives to transmute — dedicated methods for float/int/byte conversion, match for enums, and #[repr(transparent)] for newtypes**

```rust
// Safer alternatives to transmute

// 1. Float to bits (use dedicated methods)
let bits: u32 = 1.0f32.to_bits();
let float: f32 = f32::from_bits(bits);
assert_eq!(float, 1.0);

// 2. Integer to bytes (use to_ne_bytes)
let bytes: [u8; 4] = 0xDEADBEEFu32.to_ne_bytes();
let value: u32 = u32::from_ne_bytes(bytes);
assert_eq!(value, 0xDEADBEEF);

// 3. Enum with known variants
#[repr(u8)]
enum Color { Red = 0, Green = 1, Blue = 2 }

fn color_from_u8(v: u8) -> Option<Color> {
    match v {
        0 => Some(Color::Red),
        1 => Some(Color::Green),
        2 => Some(Color::Blue),
        _ => None, // Safe! No transmute needed
    }
}

// 4. #[repr(transparent)] for newtype transmute
#[repr(transparent)]
struct Meters(f64);

// SAFETY: #[repr(transparent)] guarantees same layout as f64
let distance: Meters = unsafe { std::mem::transmute(42.0f64) };
assert_eq!(distance.0, 42.0);
```


## Resources

- [std::mem::transmute](https://doc.rust-lang.org/std/mem/fn.transmute.html) — API reference for transmute with extensive examples of valid and invalid uses

---

> 📘 *This lesson is part of the [Unsafe Rust & FFI](https://stanza.dev/courses/rust-unsafe) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*