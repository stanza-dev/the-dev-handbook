---
source_course: "rust-unsafe"
source_lesson: "rust-unsafe-extern-block"
---

# Declaring Foreign Functions

```rust
extern "C" {
    fn abs(input: i32) -> i32;
    fn strlen(s: *const i8) -> usize;
    fn printf(format: *const i8, ...) -> i32;
}

fn main() {
    unsafe {
        println!("abs(-5) = {}", abs(-5));
    }
}
```

## ABI: Application Binary Interface

```rust
// "C" - C calling convention (most common)
extern "C" { ... }

// "system" - Platform's native ABI (Windows: stdcall, others: C)
extern "system" { ... }

// "Rust" - Rust ABI (default, not for FFI)
extern "Rust" { ... }
```

## Linking to Libraries

```rust
// In build.rs or Cargo.toml

// Cargo.toml
[build-dependencies]
cc = "1.0"

// build.rs
fn main() {
    // Link to system library
    println!("cargo:rustc-link-lib=ssl");
    println!("cargo:rustc-link-lib=crypto");
    
    // Or build C code
    cc::Build::new()
        .file("src/native.c")
        .compile("native");
}
```

## Using the libc Crate

```rust
use libc::{c_int, c_char, size_t};

extern "C" {
    fn getenv(name: *const c_char) -> *mut c_char;
    fn malloc(size: size_t) -> *mut libc::c_void;
    fn free(ptr: *mut libc::c_void);
}
```

See [FFI](https://doc.rust-lang.org/nomicon/ffi.html).

## Code Examples

**Safe wrapper for getenv**

```rust
use std::ffi::{CStr, CString};
use libc::{c_char, c_int};

extern "C" {
    fn getenv(name: *const c_char) -> *mut c_char;
}

fn get_env_var(name: &str) -> Option<String> {
    let name_c = CString::new(name).ok()?;
    
    unsafe {
        let ptr = getenv(name_c.as_ptr());
        if ptr.is_null() {
            None
        } else {
            // SAFETY: getenv returns valid UTF-8 (usually)
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

- [FFI](https://doc.rust-lang.org/nomicon/ffi.html) — Rustonomicon FFI chapter

---

> 📘 *This lesson is part of the [Unsafe Rust & FFI](https://stanza.dev/courses/rust-unsafe) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*