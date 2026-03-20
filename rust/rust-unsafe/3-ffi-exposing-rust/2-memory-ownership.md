---
source_course: "rust-unsafe"
source_lesson: "rust-unsafe-memory-ownership"
---

# Memory Ownership Across FFI

## Introduction
Rust and C have fundamentally different memory management models. Rust uses ownership and RAII; C uses manual `malloc`/`free`. When passing heap-allocated data across the FFI boundary, you must ensure the correct allocator frees the memory. The golden rule: memory should be freed by the same allocator that created it.

## Key Concepts
- **`Box::into_raw`**: Consumes a `Box<T>` and returns a raw pointer, preventing Rust from running the destructor.
- **`Box::from_raw`**: Reconstructs a `Box<T>` from a raw pointer, allowing Rust to drop it.
- **Opaque handle pattern**: Expose complex Rust types to C as opaque pointers with create/use/destroy functions.
- **Allocator mismatch**: Freeing memory with the wrong allocator (e.g., Rust-allocated memory with C's `free()`) is undefined behavior.

## Real World Context
Every Rust library that returns heap data to C — strings, structs, buffers — must solve the ownership problem. Libraries like `rusqlite`, `ring`, and game engines use the opaque handle pattern extensively.

## Deep Dive

### The Problem: Allocator Mismatch

Rust and C may use different allocators. Freeing Rust memory with C's `free()` is undefined behavior:

```rust
// BAD: Who frees this? C might use free(), but Rust used its own allocator!
#[unsafe(no_mangle)]
pub extern "C" fn get_data() -> *mut u8 {
    let data = vec![1u8, 2, 3];
    Box::into_raw(data.into_boxed_slice()) as *mut u8
    // Caller has no way to correctly free this!
}
```

### The Solution: Paired Create/Free Functions

Always provide a Rust-side free function:

```rust
#[unsafe(no_mangle)]
pub extern "C" fn create_buffer(size: usize) -> *mut u8 {
    let buffer = vec![0u8; size].into_boxed_slice();
    Box::into_raw(buffer) as *mut u8
}

#[unsafe(no_mangle)]
pub extern "C" fn free_buffer(ptr: *mut u8, size: usize) {
    if !ptr.is_null() {
        // SAFETY: ptr was created by create_buffer with the given size
        unsafe {
            let _ = Box::from_raw(std::slice::from_raw_parts_mut(ptr, size));
        }
    }
}
```

The C side calls `create_buffer` to allocate and `free_buffer` to deallocate. Both use Rust's allocator.

### The Opaque Handle Pattern

For complex Rust types, expose them as opaque pointers:

```rust
pub struct Database {
    connection: String,
    pool_size: usize,
}

#[unsafe(no_mangle)]
pub extern "C" fn db_open(conn: *const libc::c_char) -> *mut Database {
    let conn_str = unsafe {
        // SAFETY: caller provides valid C string
        std::ffi::CStr::from_ptr(conn).to_str().unwrap()
    };
    Box::into_raw(Box::new(Database {
        connection: conn_str.to_string(),
        pool_size: 10,
    }))
}

#[unsafe(no_mangle)]
pub extern "C" fn db_close(db: *mut Database) {
    if !db.is_null() {
        // SAFETY: db was created by db_open via Box::into_raw
        unsafe { drop(Box::from_raw(db)); }
    }
}
```

The C header declares the type as opaque:

```c
typedef struct Database Database;
Database* db_open(const char* conn);
void db_close(Database* db);
```

C code can only use the pointer through the provided API — it cannot access the struct fields.

## Common Pitfalls
1. **Calling `free()` on Rust-allocated memory** — This is UB. Always provide a Rust-side deallocation function.
2. **Forgetting `Box::from_raw` reconstructs ownership** — After `Box::from_raw`, the `Box` will drop the value. Do not use the raw pointer afterward.
3. **Double-freeing opaque handles** — Document clearly that the C side must call the destroy function exactly once. Set the pointer to NULL after freeing.

## Best Practices
1. **Always pair create/destroy functions** — For every function that returns an owned pointer, provide a matching free function.
2. **Use the opaque handle pattern** — Never expose Rust struct internals to C. Always use opaque pointers.
3. **Null-check every pointer** — Defensive null checks in your free functions prevent crashes from double-frees.

## Summary
- Memory must be freed by the same allocator that created it.
- Use `Box::into_raw` to transfer ownership to C and `Box::from_raw` to reclaim it.
- The opaque handle pattern exposes Rust types to C as opaque pointers.
- Always provide paired create/destroy functions.
- Null-check every pointer in free functions to prevent crashes.

## Code Examples

**Complete opaque handle pattern — create, use, and destroy a Rust HashMap through C-compatible functions using #[unsafe(no_mangle)] (Edition 2024)**

```rust
use std::os::raw::c_char;
use std::ffi::CStr;
use std::collections::HashMap;

pub struct Config {
    values: HashMap<String, String>,
}

pub type ConfigHandle = *mut Config;

#[unsafe(no_mangle)]
pub extern "C" fn config_new() -> ConfigHandle {
    Box::into_raw(Box::new(Config {
        values: HashMap::new(),
    }))
}

#[unsafe(no_mangle)]
pub extern "C" fn config_set(
    handle: ConfigHandle,
    key: *const c_char,
    value: *const c_char,
) -> bool {
    if handle.is_null() || key.is_null() || value.is_null() {
        return false;
    }

    // SAFETY: handle is valid (from config_new), key/value are valid C strings
    let config = unsafe { &mut *handle };
    let key = unsafe { CStr::from_ptr(key).to_str().ok() };
    let value = unsafe { CStr::from_ptr(value).to_str().ok() };

    match (key, value) {
        (Some(k), Some(v)) => {
            config.values.insert(k.to_string(), v.to_string());
            true
        }
        _ => false,
    }
}

#[unsafe(no_mangle)]
pub extern "C" fn config_free(handle: ConfigHandle) {
    if !handle.is_null() {
        // SAFETY: handle was created by config_new
        unsafe { drop(Box::from_raw(handle)); }
    }
}
```


## Resources

- [FFI Patterns — The Rustonomicon](https://doc.rust-lang.org/nomicon/ffi.html#the-nullable-pointer-optimization) — FFI patterns including opaque types and nullable pointer optimization

---

> 📘 *This lesson is part of the [Unsafe Rust & FFI](https://stanza.dev/courses/rust-unsafe) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*