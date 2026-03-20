---
source_course: "rust-unsafe"
source_lesson: "rust-unsafe-type-mapping"
---

# Type Mapping: C to Rust

## Introduction
C and Rust have different type systems, memory layouts, and string representations. Correctly mapping between them is crucial for FFI — a single wrong type can cause undefined behavior, memory corruption, or crashes. This lesson covers the complete set of type mappings you need for safe interop.

## Key Concepts
- **`#[repr(C)]`**: Attribute that forces Rust to lay out a struct's fields in the same order and alignment as C.
- **`CStr` / `CString`**: Rust types for working with null-terminated C strings.
- **`c_void`**: Rust's equivalent of C's `void *`, an opaque pointer type.
- **`NonNull<T>`**: A pointer wrapper that guarantees non-null, useful for FFI return types.

## Real World Context
Every C library binding requires type mapping. If you use `rusqlite` (SQLite), `openssl` (TLS), or any `-sys` crate, the underlying bindings map C types to Rust. Getting this wrong leads to memory corruption that is extremely hard to debug.

## Deep Dive

### Primitive Type Mappings

Use the `libc` crate for portable type aliases:

| C Type | Rust Type | `libc` alias |
|--------|-----------|------|---|
| `char` | `i8` or `u8` | `c_char` |
| `short` | `i16` | `c_short` |
| `int` | `i32` | `c_int` |
| `long` | `i32` or `i64` | `c_long` |
| `long long` | `i64` | `c_longlong` |
| `size_t` | `usize` | `size_t` |
| `void *` | `*mut c_void` | — |
| `float` | `f32` | `c_float` |
| `double` | `f64` | `c_double` |

Do not hardcode `i32` for `int` — use `c_int`, because the sizes can differ across platforms.

### String Conversion

C strings are null-terminated byte arrays. Rust strings are length-prefixed UTF-8. Converting between them requires care:

```rust
use std::ffi::{CStr, CString};

// Rust -> C: Use CString (allocates and appends null byte)
let rust_str = "hello";
let c_string = CString::new(rust_str).expect("interior null byte");
let c_ptr: *const i8 = c_string.as_ptr();
// IMPORTANT: c_ptr is only valid while c_string is alive!

// C -> Rust: Use CStr (borrows from a C pointer)
unsafe {
    let c_str: &CStr = CStr::from_ptr(c_ptr);
    let rust_str: &str = c_str.to_str().expect("invalid UTF-8");
}
```

The most common bug is dropping `CString` before the C function uses the pointer.

### Struct Layout with `#[repr(C)]`

By default, Rust reorders and pads struct fields for optimization. `#[repr(C)]` forces C-compatible layout:

```rust
use libc::c_int;

// C: struct Point { int x; int y; };
#[repr(C)]
struct Point {
    x: c_int,
    y: c_int,
}
```

Without `#[repr(C)]`, passing this struct to C would read garbage values.

### Nullable Pointers with NonNull

For C functions that might return NULL, use `Option<NonNull<T>>` for type safety:

```rust
use std::ptr::NonNull;

unsafe extern "C" {
    fn find_item(id: i32) -> *mut Item;
}

// Safe wrapper returns Option instead of raw pointer
fn find(id: i32) -> Option<NonNull<Item>> {
    // SAFETY: find_item is a valid C function
    unsafe { NonNull::new(find_item(id)) }
}
```

This eliminates an entire class of null pointer bugs.

## Common Pitfalls
1. **Dropping CString before the pointer is used** — `CString::new("hello").unwrap().as_ptr()` is a dangling pointer because the `CString` is dropped immediately. Bind it to a variable first.
2. **Forgetting `#[repr(C)]`** — Without it, Rust may reorder fields, causing C to read the wrong values. This is especially dangerous because it can appear to work in debug mode but fail in release.
3. **Using `String` instead of `CString`** — Rust `String` is not null-terminated. Passing it as a C string reads past the end of the buffer.

## Best Practices
1. **Always use `libc` type aliases** — They handle platform differences in type sizes.
2. **Use `CStr::to_str()` with proper error handling** — C strings may contain invalid UTF-8. Use `to_string_lossy()` when you can tolerate replacement characters.
3. **Keep CString alive for the duration of the FFI call** — Bind to a named variable, not a temporary.

## Summary
- Use `libc` type aliases (`c_int`, `c_char`, `size_t`) for portable FFI.
- Convert strings with `CString` (Rust to C) and `CStr` (C to Rust).
- Apply `#[repr(C)]` to all structs passed across the FFI boundary.
- Use `NonNull<T>` and `Option` instead of raw nullable pointers.
- Always keep `CString` alive while its pointer is in use.

## Code Examples

**Struct and string FFI with proper lifetime management — shows #[repr(C)], CString lifetime, and the unsafe extern block (Edition 2024)**

```rust
use std::ffi::{CStr, CString};
use std::os::raw::{c_char, c_int};

// C header:
// struct Person {
//     const char* name;
//     int age;
// };
// void greet(const struct Person* p);

#[repr(C)]
struct Person {
    name: *const c_char,
    age: c_int,
}

unsafe extern "C" {
    fn greet(person: *const Person);
}

/// Safe wrapper that handles string conversion and lifetime management.
fn greet_person(name: &str, age: i32) {
    // CString must outlive the FFI call!
    let name_c = CString::new(name).expect("name contained null byte");

    let person = Person {
        name: name_c.as_ptr(),
        age: age as c_int,
    };

    // SAFETY: person.name points to valid null-terminated string
    // (name_c is alive until end of this function)
    unsafe {
        greet(&person);
    }
    // name_c dropped here — after greet() has returned
}
```


## Resources

- [FFI — The Rustonomicon](https://doc.rust-lang.org/nomicon/ffi.html) — Detailed FFI guide covering type mappings, calling conventions, and common patterns

---

> 📘 *This lesson is part of the [Unsafe Rust & FFI](https://stanza.dev/courses/rust-unsafe) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*