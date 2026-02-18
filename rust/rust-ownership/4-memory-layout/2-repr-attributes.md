---
source_course: "rust-ownership"
source_lesson: "rust-ownership-repr-attributes"
---

# Controlling Layout with repr

## repr(C): C-Compatible Layout

Guarantees C ABI compatibility - fields in declaration order with C padding rules:

```rust
#[repr(C)]
struct CCompatible {
    a: u8,   // 1 byte
    // 3 bytes padding
    b: u32,  // 4 bytes
    c: u8,   // 1 byte
    // 3 bytes padding
}  // Total: 12 bytes, exactly as C would layout
```

## repr(packed): No Padding

```rust
#[repr(packed)]
struct Packed {
    a: u8,
    b: u32,
    c: u8,
}  // Total: 6 bytes (no padding)
```

⚠️ **Warning:** Packed structs may cause unaligned access, which is slow or crashes on some architectures.

## repr(align(N)): Force Alignment

```rust
#[repr(align(64))]
struct CacheLineAligned {
    data: [u8; 64],
}

assert_eq!(std::mem::align_of::<CacheLineAligned>(), 64);
```

## repr(transparent): Single-Field Wrapper

```rust
#[repr(transparent)]
struct Meters(f64);

// Guaranteed same layout as f64
// Safe to transmute between them
```

## repr for Enums

```rust
#[repr(u8)]
enum Status {
    Ok = 0,
    Error = 1,
    Unknown = 255,
}

assert_eq!(std::mem::size_of::<Status>(), 1);
```

## Code Examples

**FFI-compatible struct**

```rust
// FFI struct for C interop
#[repr(C)]
pub struct Point {
    pub x: f64,
    pub y: f64,
}

// Corresponding C:
// typedef struct { double x; double y; } Point;

extern "C" {
    fn process_point(p: *const Point);
}

fn main() {
    let p = Point { x: 1.0, y: 2.0 };
    unsafe { process_point(&p); }
}
```


## Resources

- [Type Layout](https://doc.rust-lang.org/reference/type-layout.html) — Complete reference for type layout

---

> 📘 *This lesson is part of the [Rust Ownership & Memory Model](https://stanza.dev/courses/rust-ownership) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*