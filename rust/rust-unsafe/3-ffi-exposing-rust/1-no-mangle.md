---
source_course: "rust-unsafe"
source_lesson: "rust-unsafe-no-mangle"
---

# #[unsafe(no_mangle)] & extern Functions

## Introduction
To call Rust functions from C, you need two things: a stable symbol name and a C-compatible calling convention. Rust's name mangling produces symbols like `_ZN7mycrate3add17h1234abcdE`, which C cannot link against. The `#[unsafe(no_mangle)]` attribute (Edition 2024 syntax) and `extern "C"` fix this.

## Key Concepts
- **Name mangling**: The compiler encodes type information into function names to support overloading and namespacing. C does not do this.
- **`#[unsafe(no_mangle)]`**: Tells the compiler to use the exact function name as the symbol, without mangling. In Edition 2024, the `unsafe()` wrapper is required because disabling mangling can cause symbol collisions.
- **`extern "C"`**: Specifies the C calling convention for the function.
- **`cdylib` / `staticlib`**: Crate types that produce shared or static libraries usable from C.

## Real World Context
Any Rust library consumed by C, Python, Ruby, or other languages through FFI needs `#[unsafe(no_mangle)]` and `extern "C"`. Projects like Firefox (Servo), Dropbox, and Discord expose Rust code to other languages this way.

## Deep Dive

### Basic Exported Function (Edition 2024)

In Edition 2024, `#[no_mangle]` must be wrapped in `#[unsafe(...)]` to acknowledge it is an unsafe operation:

```rust
#[unsafe(no_mangle)]
pub extern "C" fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

The `#[unsafe(no_mangle)]` is required because:
1. It exposes a global symbol that could collide with other symbols.
2. It commits to a stable ABI that Rust normally does not guarantee.

Without `#[unsafe(no_mangle)]`, the function name would be mangled:

```rust
// Without: symbol is _ZN7mycrate3add17h1234567890abcdefE
// With:    symbol is add
```

### Building a C Library

Configure your `Cargo.toml` to output a C-compatible library:

```toml
[lib]
crate-type = ["cdylib", "staticlib"]

# cdylib = shared library (.so on Linux, .dylib on macOS, .dll on Windows)
# staticlib = static library (.a on Unix, .lib on Windows)
```

`cdylib` is for shared/dynamic libraries. `staticlib` is for static linking.

### Writing the C Header

C code needs a header file declaring the Rust functions:

```c
/* mylib.h */
#ifndef MYLIB_H
#define MYLIB_H

#include <stdint.h>

int32_t add(int32_t a, int32_t b);

#endif
```

For large APIs, use `cbindgen` to auto-generate headers.

### cbindgen: Automatic Header Generation

The `cbindgen` crate generates C headers from your Rust source:

```toml
# Cargo.toml
[build-dependencies]
cbindgen = "0.26"
```

```rust
// build.rs
fn main() {
    cbindgen::Builder::new()
        .with_crate(".")
        .generate()
        .expect("Unable to generate bindings")
        .write_to_file("include/mylib.h");
}
```

This reads your `#[unsafe(no_mangle)]` functions and produces a correct C header automatically.

## Common Pitfalls
1. **Using `#[no_mangle]` instead of `#[unsafe(no_mangle)]` in Edition 2024** — The bare form is a compile error. Always wrap in `unsafe()`.
2. **Forgetting `pub`** — Without `pub`, the symbol may not be exported from the shared library.
3. **Returning Rust-only types** — Types like `String`, `Vec`, or `Result` have no C representation. Only return `#[repr(C)]` structs, primitive types, or raw pointers.

## Best Practices
1. **Use `cbindgen`** — Hand-written headers get out of sync. Auto-generate them.
2. **Prefix exported symbols** — Use a library prefix like `mylib_add` to avoid name collisions with other libraries.
3. **Document the C API** — Add comments in both the Rust source and the generated header explaining ownership and thread-safety guarantees.

## Summary
- Edition 2024 requires `#[unsafe(no_mangle)]` instead of `#[no_mangle]`.
- Combine `#[unsafe(no_mangle)]` with `extern "C"` to create C-callable functions.
- Set `crate-type = ["cdylib"]` in Cargo.toml for shared libraries.
- Use `cbindgen` to auto-generate C headers.
- Only return C-compatible types: primitives, `#[repr(C)]` structs, and raw pointers.

## Code Examples

**String FFI with memory management — shows #[unsafe(no_mangle)] (Edition 2024), CString ownership transfer, and paired alloc/free functions**

```rust
use std::ffi::{CStr, CString};
use std::os::raw::c_char;

/// Concatenate two C strings. Caller must free the result with free_string.
#[unsafe(no_mangle)]
pub extern "C" fn concat_strings(
    a: *const c_char,
    b: *const c_char,
) -> *mut c_char {
    let a_str = unsafe {
        if a.is_null() { return std::ptr::null_mut(); }
        // SAFETY: caller guarantees a is a valid C string
        CStr::from_ptr(a).to_str().unwrap_or("")
    };

    let b_str = unsafe {
        if b.is_null() { return std::ptr::null_mut(); }
        // SAFETY: caller guarantees b is a valid C string
        CStr::from_ptr(b).to_str().unwrap_or("")
    };

    let result = format!("{a_str}{b_str}");
    CString::new(result)
        .map(|s| s.into_raw())
        .unwrap_or(std::ptr::null_mut())
}

/// Free a string returned by concat_strings.
#[unsafe(no_mangle)]
pub extern "C" fn free_string(s: *mut c_char) {
    if !s.is_null() {
        // SAFETY: s was created by CString::into_raw in concat_strings
        unsafe { drop(CString::from_raw(s)); }
    }
}
```


## Resources

- [cbindgen — C/C++ Header Generator](https://github.com/mozilla/cbindgen) — Automatically generate C/C++ headers from Rust source code
- [FFI — The Rustonomicon](https://doc.rust-lang.org/nomicon/ffi.html) — Official guide to Rust's Foreign Function Interface

---

> 📘 *This lesson is part of the [Unsafe Rust & FFI](https://stanza.dev/courses/rust-unsafe) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*