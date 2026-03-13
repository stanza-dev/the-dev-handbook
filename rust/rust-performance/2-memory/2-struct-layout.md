---
source_course: "rust-performance"
source_lesson: "rust-perf-struct-layout"
---

# Struct Layout & Packing

## Introduction

Rust's default struct layout reorders fields for optimal alignment and minimal padding. Understanding how the compiler lays out your types lets you minimize memory usage and improve cache utilization.

## Key Concepts

### Default Layout

Rust is free to reorder fields to minimize padding:

```rust
struct Example {
    a: u8,   // 1 byte
    b: u64,  // 8 bytes
    c: u16,  // 2 bytes
}
// Rust reorders to: b(8) + c(2) + a(1) + padding(5) = 16 bytes
```

### `#[repr(C)]`: C-Compatible Layout

```rust
#[repr(C)]
struct CExample {
    a: u8,   // offset 0, +7 padding
    b: u64,  // offset 8
    c: u16,  // offset 16, +6 padding
}  // Total: 24 bytes (worse!)
```

### `#[repr(packed)]`: No Padding

```rust
#[repr(packed)]
struct Packed {
    a: u8,
    b: u64,
    c: u16,
}  // Total: 11 bytes, but unaligned access is slow
```

## Real World Context

Network protocols and file formats use `repr(C)` for FFI compatibility. Game engines carefully control struct sizes to fit more entities in cache. Serialization libraries rely on predictable layouts.

## Deep Dive

### Checking Size and Alignment

```rust
use std::mem::{size_of, align_of};

println!("Size: {}", size_of::<Example>());   // 16
println!("Align: {}", align_of::<Example>()); // 8
```

### Null Pointer Optimization

Rust uses niche optimization: `Option<&T>` is the same size as `&T` because null is used for `None`:

```rust
assert_eq!(size_of::<&u64>(), 8);
assert_eq!(size_of::<Option<&u64>>(), 8); // Free Option!

// Also works for Box, NonNull, NonZeroU64, etc.
assert_eq!(size_of::<Option<Box<u64>>>(), 8);
```

### Optimizing Enum Size

```rust
// Large variant makes the whole enum large
enum Message {
    Quit,
    Text(String),           // 24 bytes
    Data([u8; 1024]),       // 1024 bytes!
}
// size_of::<Message>() = 1032 (discriminant + largest variant)

// Fix: Box the large variant
enum MessageOptimized {
    Quit,
    Text(String),
    Data(Box<[u8; 1024]>),  // 8 bytes (pointer)
}
// size_of::<MessageOptimized>() = 32
```

## Common Pitfalls

- Using `#[repr(packed)]` without understanding the performance cost — unaligned access is slow on most architectures and references to packed fields are unsafe.
- Assuming field order in default layout — Rust provides no guarantees.
- Creating enums with one disproportionately large variant — Box it instead.

## Best Practices

- Let the compiler optimize layout (default repr) unless you need FFI compatibility.
- Use `size_of` and `align_of` to verify your assumptions.
- Box large enum variants to keep the enum small.
- Use `NonZeroU*` types when zero is not a valid value to enable niche optimization.

## Summary

Rust reorders struct fields to minimize padding by default. Use `repr(C)` for FFI, avoid `repr(packed)` unless necessary, leverage null pointer optimization with `Option`, and Box large enum variants. Always verify sizes with `std::mem::size_of`.

## Code Examples

**Null pointer optimization and enum size reduction**

```rust
use std::mem::size_of;
use std::num::NonZeroU64;

// Null pointer optimization
assert_eq!(size_of::<&u64>(), 8);
assert_eq!(size_of::<Option<&u64>>(), 8); // Same size!
assert_eq!(size_of::<Box<u64>>(), 8);
assert_eq!(size_of::<Option<Box<u64>>>(), 8); // Same size!
assert_eq!(size_of::<Option<NonZeroU64>>(), 8); // Same size!

// Enum size optimization
enum Large {
    A,
    B([u8; 1024]),
}
enum Small {
    A,
    B(Box<[u8; 1024]>),
}
println!("Large: {} bytes", size_of::<Large>());  // 1028
println!("Small: {} bytes", size_of::<Small>());  // 16
```


## Resources

- [Rust Reference — Type Layout](https://doc.rust-lang.org/reference/type-layout.html) — Official documentation on Rust type layout rules

---

> 📘 *This lesson is part of the [Rust Performance](https://stanza.dev/courses/rust-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*