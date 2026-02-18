---
source_course: "rust-performance"
source_lesson: "rust-perf-struct-layout"
---

# Struct Layout in Rust

## Default Layout

Rust reorders fields for optimal alignment:

```rust
// You write:
struct Example {
    a: u8,   // 1 byte
    b: u64,  // 8 bytes
    c: u16,  // 2 bytes
}

// Rust may reorder to:
// b: u64 (offset 0)
// c: u16 (offset 8)
// a: u8  (offset 10)
// padding: 5 bytes
// Total: 16 bytes (aligned to 8)
```

## #[repr(C)]: C-Compatible Layout

```rust
#[repr(C)]
struct CExample {
    a: u8,   // offset 0, 1 byte + 7 padding
    b: u64,  // offset 8, 8 bytes
    c: u16,  // offset 16, 2 bytes + 6 padding
}  // Total: 24 bytes (worse!)
```

## #[repr(packed)]: No Padding

```rust
#[repr(packed)]
struct Packed {
    a: u8,
    b: u64,
    c: u16,
}  // Total: 11 bytes

// Warning: Unaligned access is slow!
// And taking references to packed fields is unsafe
let x = &packed.b;  // Compile error without unsafe!
```

## Checking Size and Alignment

```rust
use std::mem::{size_of, align_of};

println!("Size: {}", size_of::<Example>());
println!("Align: {}", align_of::<Example>());

// Or at compile time:
const SIZE: usize = size_of::<Example>();
```

## Optimizing Struct Size

```rust
// Bad: 24 bytes with padding
struct Bad {
    a: u8,
    b: u64,
    c: u8,
}

// Good: 16 bytes, group by alignment
struct Good {
    b: u64,  // 8-byte aligned first
    a: u8,
    c: u8,
}  // 2 bytes padding at end
```

## Code Examples

**Size optimization**

```rust
use std::mem::{size_of, align_of};

// Enum size optimization
// Null pointer optimization makes Option<&T> same size as &T!

println!("&u8: {}", size_of::<&u8>());           // 8
println!("Option<&u8>: {}", size_of::<Option<&u8>>()); // 8 (not 16!)

// Box, NonNull, etc also get this optimization
println!("Box<u8>: {}", size_of::<Box<u8>>());     // 8
println!("Option<Box<u8>>: {}", size_of::<Option<Box<u8>>>()); // 8

// NonZero types enable similar optimization
use std::num::NonZeroU64;
println!("NonZeroU64: {}", size_of::<NonZeroU64>()); // 8
println!("Option<NonZeroU64>: {}", size_of::<Option<NonZeroU64>>()); // 8
```


---

> 📘 *This lesson is part of the [Rust Performance](https://stanza.dev/courses/rust-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*