---
source_course: "rust-ownership"
source_lesson: "rust-ownership-struct-layout"
---

# Struct Memory Layout

## Introduction
Understanding how Rust lays out structs in memory is essential for writing cache-friendly code, interfacing with C libraries, and predicting memory usage. Unlike C, Rust's default layout may reorder fields to minimize wasted padding bytes.

## Key Concepts
- **Alignment**: A type's alignment is the number that its starting memory address must be divisible by. For example, a u64 has 8-byte alignment.
- **Padding**: Extra bytes inserted between fields to satisfy alignment requirements.
- **Zero-Sized Type (ZST)**: A type with size 0, such as `()` or an empty struct, which takes no space in memory.

## Real World Context
When you serialize data, write network protocols, or share memory with C code, field layout matters. A struct with poor field ordering can waste significant memory due to padding — relevant when allocating millions of instances. Cache-line alignment can also affect performance in hot loops.

## Deep Dive
Every type has a size and alignment. The alignment determines where in memory the value can be placed:

```rust
use std::mem::{size_of, align_of};

assert_eq!(align_of::<u8>(), 1);
assert_eq!(align_of::<u32>(), 4);
assert_eq!(align_of::<u64>(), 8);
```

When fields have different alignments, the compiler inserts padding bytes to satisfy each field's alignment:

```rust
// With repr(C), fields stay in declaration order:
#[repr(C)]
struct Unoptimized {
    a: u8,   // 1 byte
    // 7 bytes padding (to align b to 8)
    b: u64,  // 8 bytes
    c: u8,   // 1 byte
    // 7 bytes padding (struct must be multiple of alignment)
}  // Total: 24 bytes
```

Rust's default layout may reorder fields to reduce padding:

```rust
// Default Rust layout may reorder to:
struct Optimized {
    b: u64,  // 8 bytes
    a: u8,   // 1 byte
    c: u8,   // 1 byte
    // 6 bytes padding
}  // Total: 16 bytes — saves 8 bytes
```

Enums use space for the largest variant plus a discriminant (tag). Rust applies the null pointer optimization where possible:

```rust
use std::mem::size_of;

enum Message {
    Quit,                           // 0 bytes payload
    Move { x: i32, y: i32 },       // 8 bytes payload
    Write(String),                  // 24 bytes payload (ptr + len + cap)
    ChangeColor(u8, u8, u8),       // 3 bytes payload
}

// Size = largest variant (24 for String) + 1 byte discriminant
// + 7 bytes padding for 8-byte alignment = 32 bytes total
println!("Message size: {}", size_of::<Message>());  // 32

// Null pointer optimization: Option<Box<T>> = same size as Box<T>
assert_eq!(size_of::<Box<i32>>(), size_of::<Option<Box<i32>>>());
```

Zero-sized types take no space:

```rust
struct Marker;
assert_eq!(size_of::<Marker>(), 0);
assert_eq!(size_of::<()>(), 0);
```

## Common Pitfalls
1. **Assuming field order matches declaration order** — Rust's default layout does not guarantee field order. Only `#[repr(C)]` gives that guarantee.
2. **Ignoring padding in memory-critical code** — A struct with three u8 fields and one u64 can waste up to 50% of its memory on padding if fields are ordered poorly.

## Best Practices
1. **Use `std::mem::size_of` to check struct sizes** — Especially when designing types that will have millions of instances.
2. **Order fields from largest to smallest alignment** — This naturally minimizes padding in `#[repr(C)]` structs and gives hints to the default layout optimizer.

## Summary
- Rust may reorder struct fields to minimize padding.
- Alignment determines where types can start in memory.
- Enum size equals the largest variant plus discriminant plus padding.
- Option<Box<T>> gets null pointer optimization, costing no extra space.
- Zero-sized types take no memory.

## Code Examples

**Enum and Option sizes — the compiler uses null pointer optimization to make Option<Box<T>> zero-cost**

```rust
use std::mem::size_of;

// Enum size = largest variant + discriminant + padding
enum Message {
    Quit,                           // 0 bytes payload
    Move { x: i32, y: i32 },       // 8 bytes payload
    Write(String),                  // 24 bytes payload (ptr + len + cap)
    ChangeColor(u8, u8, u8),       // 3 bytes payload
}

// Size: 24 (String, largest variant) + 1 (discriminant)
// + 7 (padding to 8-byte alignment) = 32 bytes
println!("Message size: {}", size_of::<Message>()); // 32

// Option<Box<T>> is same size as Box<T>!
// Null pointer optimization: None uses the 0x0 address
assert_eq!(size_of::<Box<i32>>(), size_of::<Option<Box<i32>>>());
```


## Resources

- [Type Layout Reference](https://doc.rust-lang.org/reference/type-layout.html) — Official Rust reference for type layout rules

---

> 📘 *This lesson is part of the [Rust Ownership & Memory Model](https://stanza.dev/courses/rust-ownership) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*