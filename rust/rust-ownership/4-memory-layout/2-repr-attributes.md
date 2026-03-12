---
source_course: "rust-ownership"
source_lesson: "rust-ownership-repr-attributes"
---

# repr Attributes

## Introduction
Rust's default struct layout is optimized for space but not guaranteed to match any particular convention. The `repr` attribute lets you control exactly how data is laid out, which is essential for FFI, hardware registers, and cache optimization.

## Key Concepts
- **repr(C)**: Lays out fields in declaration order with C-compatible padding rules, essential for FFI.
- **repr(packed)**: Removes all padding, producing the smallest possible size at the risk of unaligned access.
- **repr(align(N))**: Forces a minimum alignment of N bytes.
- **repr(transparent)**: Guarantees a single-field struct has the same layout as its field.

## Real World Context
Any Rust code that interfaces with C (via FFI), reads binary file formats, or communicates with hardware registers needs precise control over memory layout. Game engines use repr(align) for cache-line alignment, and network protocols often require repr(C) or repr(packed) to match wire formats.

## Deep Dive
`repr(C)` guarantees C ABI compatibility with fields in declaration order:

```rust
#[repr(C)]
struct CCompatible {
    a: u8,   // 1 byte
    // 3 bytes padding
    b: u32,  // 4 bytes
    c: u8,   // 1 byte
    // 3 bytes padding
}  // Total: 12 bytes, exactly as C would lay it out
```

`repr(packed)` removes all padding, which can cause unaligned access issues:

```rust
#[repr(packed)]
struct Packed {
    a: u8,
    b: u32,
    c: u8,
}  // Total: 6 bytes (no padding)
// Warning: unaligned access can be slow or crash on some architectures
```

`repr(align(N))` forces a minimum alignment, useful for cache-line alignment:

```rust
#[repr(align(64))]
struct CacheLineAligned {
    data: [u8; 64],
}
assert_eq!(std::mem::align_of::<CacheLineAligned>(), 64);
```

`repr(transparent)` guarantees a newtype wrapper has identical layout to its inner type:

```rust
#[repr(transparent)]
struct Meters(f64);
// Guaranteed same layout as f64 — safe to transmute
```

For enums, `repr(u8)`, `repr(u16)`, etc., control the discriminant size:

```rust
#[repr(u8)]
enum Status {
    Ok = 0,
    Error = 1,
    Unknown = 255,
}
assert_eq!(std::mem::size_of::<Status>(), 1);
```

## Common Pitfalls
1. **Using repr(packed) without understanding alignment risks** — Unaligned access is undefined behavior on some architectures and slow on x86. Taking a reference to a packed field is UB in Rust.
2. **Forgetting repr(C) for FFI structs** — Without repr(C), Rust may reorder fields, causing the layout to mismatch the C definition and corrupting data across the FFI boundary.
3. **Omitting `unsafe` on `extern` blocks (2024 edition)** — The 2024 edition requires `unsafe extern "C" { ... }` instead of `extern "C" { ... }`. Declaring FFI items is inherently unsafe because the compiler cannot verify the foreign signatures are correct.

## Best Practices
1. **Always use repr(C) for FFI types** — Even if the current compiler layout happens to match C, it is not guaranteed and can change between compiler versions.
2. **Use repr(transparent) for newtypes passed across FFI** — This guarantees the wrapper has zero overhead and identical ABI to the inner type.

## Summary
- repr(C) gives C-compatible, declaration-order layout for FFI.
- repr(packed) removes padding but risks unaligned access.
- repr(align(N)) forces minimum alignment for cache optimization.
- repr(transparent) makes a single-field wrapper layout-identical to its field.
- repr(u8/u16/...) controls enum discriminant size.

## Code Examples

**FFI-compatible struct — repr(C) guarantees the layout matches the C definition. Note: the 2024 edition requires unsafe on extern blocks**

```rust
// FFI struct for C interop
#[repr(C)]
pub struct Point {
    pub x: f64,
    pub y: f64,
}

// Corresponding C:
// typedef struct { double x; double y; } Point;

unsafe extern "C" {
    fn process_point(p: *const Point);
}

fn main() {
    let p = Point { x: 1.0, y: 2.0 };
    unsafe { process_point(&p); }
}
```


## Resources

- [Type Layout](https://doc.rust-lang.org/reference/type-layout.html) — Complete reference for type layout and repr attributes

---

> 📘 *This lesson is part of the [Rust Ownership & Memory Model](https://stanza.dev/courses/rust-ownership) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*