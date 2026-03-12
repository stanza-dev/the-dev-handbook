---
source_course: "rust-ownership"
source_lesson: "rust-ownership-size-optimization"
---

# Size Optimization Techniques

## Introduction
When your program allocates millions of structs or sends data over the network, every byte matters. Rust provides several tools and patterns for minimizing type sizes, from compiler optimizations like niche filling to manual techniques like field reordering and bitpacking.

## Key Concepts
- **Niche optimization**: The compiler uses invalid bit patterns (niches) in types to store enum discriminants for free, e.g., `Option<Box<T>>` uses the null pointer as `None`.
- **Field reordering**: Rust's default layout reorders fields to minimize padding, but `repr(C)` disables this.
- **NonZero types**: Types like `NonZeroU32` guarantee a non-zero value, enabling niche optimization in `Option<NonZeroU32>`.

## Real World Context
Database engines, game entity systems, and network serialization libraries care deeply about type sizes. A 4-byte savings per record across 10 million records saves 40 MB. Understanding Rust's size optimizations helps you make informed tradeoffs.

## Deep Dive
The null pointer optimization is the most well-known niche optimization:

```rust
use std::mem::size_of;

// Box<T> cannot be null, so Option uses null as None
assert_eq!(size_of::<Box<i32>>(), size_of::<Option<Box<i32>>>());
// Both are 8 bytes on 64-bit — zero overhead!

// Same for references
assert_eq!(size_of::<&i32>(), size_of::<Option<&i32>>());
```

NonZero types enable the same optimization for integers:

```rust
use std::num::NonZeroU32;
use std::mem::size_of;

// NonZeroU32 is guaranteed non-zero, so Option uses 0 as None
assert_eq!(size_of::<NonZeroU32>(), size_of::<Option<NonZeroU32>>());
// Both are 4 bytes
```

You can check actual sizes at compile time with const assertions:

```rust
const _: () = assert!(std::mem::size_of::<MyStruct>() <= 64);
```

For complex types, consider boxing large variants to keep enum sizes small:

```rust
// Bad: the large variant inflates the enum
enum Event {
    Click { x: f64, y: f64 },
    Data([u8; 1024]),  // Forces entire enum to 1032+ bytes
}

// Better: box the large variant
enum EventCompact {
    Click { x: f64, y: f64 },
    Data(Box<[u8; 1024]>),  // Enum is ~24 bytes
}
```

## Common Pitfalls
1. **Ignoring enum variant sizes** — A single large variant inflates the size of the entire enum. All variants occupy the same amount of memory.
2. **Over-optimizing small structs** — Field reordering saves padding, but for structs allocated once, the few bytes saved are not worth the complexity.

## Best Practices
1. **Use NonZero types for IDs and counts** — They communicate the invariant to readers and enable niche optimization in Option.
2. **Box large enum variants** — If one variant is much larger than others, wrap it in Box to keep the common case small.

## Summary
- Rust optimizes Option<Box<T>> and Option<&T> to be zero-cost via niche filling.
- NonZero types enable the same optimization for integers.
- Boxing large enum variants keeps the enum size manageable.
- Use std::mem::size_of and const assertions to verify type sizes.

## Code Examples

**Niche optimization and compile-time size assertions — Option<Box<T>> costs zero extra bytes**

```rust
use std::mem::size_of;
use std::num::NonZeroU64;

// Niche optimization examples
assert_eq!(size_of::<Box<i32>>(), size_of::<Option<Box<i32>>>());
assert_eq!(size_of::<&str>(), size_of::<Option<&str>>());
assert_eq!(size_of::<NonZeroU64>(), size_of::<Option<NonZeroU64>>());

// Compile-time size check
struct Record {
    id: NonZeroU64,
    flags: u8,
    value: f64,
}
const _: () = assert!(size_of::<Record>() <= 24);
```


## Resources

- [NonZero Types](https://doc.rust-lang.org/std/num/type.NonZeroU32.html) — Official docs for NonZero integer types

---

> 📘 *This lesson is part of the [Rust Ownership & Memory Model](https://stanza.dev/courses/rust-ownership) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*