---
source_course: "rust-unsafe"
source_lesson: "rust-unsafe-no-mangle"
---

# Exposing Rust Functions to C

```rust
#[no_mangle]
pub extern "C" fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

## #[no_mangle]

Prevents Rust from "mangling" the function name:

```rust
// Without #[no_mangle]:
// Symbol: _ZN7mycrate3add17h1234567890abcdefE

// With #[no_mangle]:
// Symbol: add
```

## Building a C Library

```toml
# Cargo.toml
[lib]
crate-type = ["cdylib", "staticlib"]

# cdylib = shared library (.so, .dylib, .dll)
# staticlib = static library (.a, .lib)
```

## The Header File

```c
// mylib.h
#ifndef MYLIB_H
#define MYLIB_H

#include <stdint.h>

int32_t add(int32_t a, int32_t b);

#endif
```

## cbindgen: Generate Headers Automatically

```toml
# Cargo.toml
[build-dependencies]
cbindgen = "0.24"
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

See [cbindgen](https://github.com/mozilla/cbindgen).

## Code Examples

**String FFI with memory management**

```rust
// A complete FFI library
use std::ffi::{CStr, CString};
use std::os::raw::c_char;

/// Concatenate two strings. Caller must free the result.
#[no_mangle]
pub extern "C" fn concat_strings(
    a: *const c_char,
    b: *const c_char,
) -> *mut c_char {
    let a_str = unsafe {
        if a.is_null() { return std::ptr::null_mut(); }
        CStr::from_ptr(a).to_str().unwrap_or("")
    };
    
    let b_str = unsafe {
        if b.is_null() { return std::ptr::null_mut(); }
        CStr::from_ptr(b).to_str().unwrap_or("")
    };
    
    let result = format!("{a_str}{b_str}");
    CString::new(result)
        .map(|s| s.into_raw())
        .unwrap_or(std::ptr::null_mut())
}

/// Free a string returned by concat_strings
#[no_mangle]
pub extern "C" fn free_string(s: *mut c_char) {
    if !s.is_null() {
        unsafe { drop(CString::from_raw(s)); }
    }
}
```


## Resources

- [cbindgen](https://github.com/mozilla/cbindgen) — C header generation tool

---

> 📘 *This lesson is part of the [Unsafe Rust & FFI](https://stanza.dev/courses/rust-unsafe) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*