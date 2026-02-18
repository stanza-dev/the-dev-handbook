---
source_course: "rust-ownership"
source_lesson: "rust-ownership-struct-layout"
---

# How Rust Lays Out Structs

By default, Rust doesn't guarantee field order. It may reorder to minimize padding.

## Alignment Requirements

Types have alignment requirements - they must start at addresses divisible by their alignment:

```rust
use std::mem::{size_of, align_of};

assert_eq!(align_of::<u8>(), 1);
assert_eq!(align_of::<u32>(), 4);
assert_eq!(align_of::<u64>(), 8);
```

## Padding Example

```rust
struct Unoptimized {
    a: u8,   // 1 byte
    // 7 bytes padding (to align b)
    b: u64,  // 8 bytes
    c: u8,   // 1 byte
    // 7 bytes padding (struct alignment)
}  // Total: 24 bytes

// Rust may reorder to:
struct Optimized {
    b: u64,  // 8 bytes
    a: u8,   // 1 byte
    c: u8,   // 1 byte
    // 6 bytes padding
}  // Total: 16 bytes
```

## Checking Size and Alignment

```rust
use std::mem::{size_of, align_of};

#[derive(Debug)]
struct MyStruct {
    a: u8,
    b: u64,
    c: u8,
}

println!("Size: {}", size_of::<MyStruct>());
println!("Align: {}", align_of::<MyStruct>());
```

## Zero-Sized Types (ZSTs)

```rust
struct Empty;
struct Marker;

assert_eq!(size_of::<Empty>(), 0);
assert_eq!(size_of::<()>(), 0);
assert_eq!(size_of::<[u8; 0]>(), 0);
```

ZSTs take no space but can still be useful as markers.

See [Type Layout](https://doc.rust-lang.org/reference/type-layout.html).

## Code Examples

**Enum and Option sizes**

```rust
use std::mem::size_of;

// Enum size = largest variant + discriminant
enum Message {
    Quit,                          // 0 bytes
    Move { x: i32, y: i32 },       // 8 bytes
    Write(String),                 // 24 bytes (ptr + len + cap)
    ChangeColor(u8, u8, u8),       // 3 bytes
}

// Size = 24 (String) + 8 (discriminant alignment) = 32
println!("Message size: {}", size_of::<Message>());

// Option<Box<T>> is same size as Box<T>!
// Null pointer optimization
assert_eq!(size_of::<Box<i32>>(), size_of::<Option<Box<i32>>>());
```


---

> 📘 *This lesson is part of the [Rust Ownership & Memory Model](https://stanza.dev/courses/rust-ownership) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*