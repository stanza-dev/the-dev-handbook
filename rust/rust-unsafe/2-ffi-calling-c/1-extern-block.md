---
source_course: "rust-unsafe"
source_lesson: "rust-unsafe-extern-block"
---

# The unsafe extern Block

## Introduction
Rust can call functions written in C (and other languages) through Foreign Function Interface (FFI) declarations. Since Rust 1.85 / Edition 2024, extern blocks must be prefixed with `unsafe` to make it explicit that the compiler cannot verify the correctness of these foreign function declarations.

## Key Concepts
- **`unsafe extern` block**: A declaration block listing foreign functions. The `unsafe` prefix is required in Edition 2024.
- **ABI (Application Binary Interface)**: Specifies how functions are called at the binary level — register usage, stack layout, and calling conventions.
- **`"C"` ABI**: The most common ABI for FFI, matching the C calling convention on the target platform.
- **`"system"` ABI**: Uses `stdcall` on Windows and `"C"` on other platforms.

## Real World Context
Nearly every Rust program that interacts with the operating system uses FFI indirectly (through `libc` or `std`). If you integrate with databases (SQLite, PostgreSQL), graphics libraries (OpenGL, Vulkan), or system APIs (POSIX, Win32), you will write `unsafe extern` blocks.

## Deep Dive

### Basic extern Block (Edition 2024)

In Edition 2024, the `unsafe` keyword is required on extern blocks. This is a syntactic change that makes the unsafety more visible:

```rust
// Edition 2024: unsafe extern is required
unsafe extern "C" {
    fn abs(input: i32) -> i32;
    fn strlen(s: *const i8) -> usize;
    fn printf(format: *const i8, ...) -> i32;
}

fn main() {
    // Calling foreign functions still requires an unsafe block
    unsafe {
        println!("abs(-5) = {}", abs(-5));
    }
}
```

The `unsafe extern` tells the reader: "These function signatures are unverified — the compiler trusts us that they match the actual C signatures."

### ABI Strings

The ABI string after `extern` specifies the calling convention:

```rust
// "C" - C calling convention (most common for FFI)
unsafe extern "C" {
    fn c_function();
}

// "system" - Platform's native ABI
// Windows: stdcall, others: same as "C"
unsafe extern "system" {
    fn system_function();
}
```

Use `"C"` for most C library bindings. Use `"system"` for Windows API calls (which use `stdcall`).

### Linking to Libraries

To link against a system library, tell Cargo where to find it:

```rust
// In build.rs
fn main() {
    // Link to a system library
    println!("cargo:rustc-link-lib=ssl");
    println!("cargo:rustc-link-lib=crypto");

    // Or compile C source files
    cc::Build::new()
        .file("src/native.c")
        .compile("native");
}
```

The `cc` crate (add to `[build-dependencies]`) compiles C files and links them automatically.

### Using the `libc` Crate for Portable Types

The `libc` crate provides platform-correct type aliases:

```rust
use libc::{c_int, c_char, size_t, c_void};

unsafe extern "C" {
    fn getenv(name: *const c_char) -> *mut c_char;
    fn malloc(size: size_t) -> *mut c_void;
    fn free(ptr: *mut c_void);
}
```

Using `c_int` instead of `i32` ensures your bindings work correctly on platforms where C's `int` is not 32 bits.

## Common Pitfalls
1. **Forgetting `unsafe` on extern blocks in Edition 2024** — Plain `extern "C" { ... }` is a compile error in Edition 2024. Always write `unsafe extern "C" { ... }`.
2. **Mismatched function signatures** — If your Rust declaration doesn't match the actual C function, you get undefined behavior at runtime. Use `bindgen` to auto-generate correct bindings.
3. **Forgetting to link the library** — Declaring an `unsafe extern` block without linking the library causes linker errors. Use `build.rs` or `#[link(name = "...")]`.

## Best Practices
1. **Use `bindgen` for large C APIs** — Manually writing extern blocks is error-prone. The `bindgen` crate generates Rust bindings from C headers.
2. **Wrap every foreign call in a safe function** — Never expose raw FFI functions in your public API. Always create safe wrappers that handle null checks, string conversion, and error codes.
3. **Use `libc` types for portability** — `c_int`, `c_char`, and `size_t` match the C types on the target platform.

## Summary
- Edition 2024 requires `unsafe extern "C"` instead of plain `extern "C"`.
- The `"C"` ABI is the standard for FFI; `"system"` adds Windows `stdcall` support.
- Use `build.rs` to link against system libraries or compile C source files.
- Use the `libc` crate for portable C type aliases.
- Always wrap foreign function calls in safe Rust functions.

## Code Examples

**A safe Rust wrapper around the C getenv function — demonstrates unsafe extern block syntax (Edition 2024), CString conversion, null checking, and the safe wrapper pattern**

```rust
use std::ffi::{CStr, CString};
use libc::c_char;

// Edition 2024: unsafe extern required
unsafe extern "C" {
    fn getenv(name: *const c_char) -> *mut c_char;
}

/// Safely retrieves an environment variable.
/// Returns None if the variable is not set or contains invalid UTF-8.
fn get_env_var(name: &str) -> Option<String> {
    let name_c = CString::new(name).ok()?;

    // SAFETY: getenv is a standard C function. We pass a valid
    // null-terminated string. The returned pointer is either null
    // or points to a valid C string owned by the environment.
    unsafe {
        let ptr = getenv(name_c.as_ptr());
        if ptr.is_null() {
            None
        } else {
            Some(CStr::from_ptr(ptr).to_string_lossy().into_owned())
        }
    }
}

fn main() {
    if let Some(path) = get_env_var("PATH") {
        println!("PATH = {path}");
    }
}
```


## Resources

- [FFI — The Rustonomicon](https://doc.rust-lang.org/nomicon/ffi.html) — Comprehensive guide to Rust's Foreign Function Interface
- [bindgen User Guide](https://rust-lang.github.io/rust-bindgen/) — Automatically generate Rust FFI bindings from C headers

---

> 📘 *This lesson is part of the [Unsafe Rust & FFI](https://stanza.dev/courses/rust-unsafe) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*