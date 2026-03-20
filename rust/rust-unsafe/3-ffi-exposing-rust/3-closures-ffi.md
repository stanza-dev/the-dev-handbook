---
source_course: "rust-unsafe"
source_lesson: "rust-unsafe-callbacks-closures-ffi"
---

# Callbacks & Closures Across FFI

## Introduction
Many C libraries accept callback function pointers for event handling, iteration, or customization. Passing Rust functions (and especially closures) as C callbacks requires careful handling of function pointer types, lifetimes, and data pointers.

## Key Concepts
- **`extern "C" fn`**: A function pointer type with C calling convention, suitable for passing to C.
- **User data pointer**: A `*mut c_void` that C passes back to callbacks, allowing you to carry Rust state.
- **Trampoline function**: An `extern "C"` function that converts the user data pointer back to a Rust reference and calls a closure.

## Real World Context
C libraries like `libcurl`, `sqlite3`, `libuv`, and POSIX `signal()` all use callbacks. Any Rust wrapper for these libraries must solve the closure-to-C-callback problem.

## Deep Dive

### Simple Function Pointer Callback

The simplest case is passing a Rust function (not a closure) as a callback:

```rust
unsafe extern "C" {
    fn register_callback(cb: extern "C" fn(i32));
}

extern "C" fn my_callback(value: i32) {
    println!("Got value: {value}");
}

fn main() {
    // SAFETY: my_callback is a valid extern "C" function
    unsafe { register_callback(my_callback); }
}
```

This works because `extern "C" fn` is directly compatible with C function pointers.

### Closures via User Data Pointer

C callbacks often accept a `void *user_data` that is passed back to the callback. This lets you carry Rust state:

```rust
unsafe extern "C" {
    fn set_handler(
        cb: extern "C" fn(*mut libc::c_void, i32),
        user_data: *mut libc::c_void,
    );
}

// Trampoline: converts void* back to our closure
extern "C" fn trampoline(data: *mut libc::c_void, value: i32) {
    // SAFETY: data points to a valid Box<dyn FnMut(i32)>
    let closure = unsafe { &mut *(data as *mut Box<dyn FnMut(i32)>) };
    closure(value);
}

fn register_closure(mut f: Box<dyn FnMut(i32)>) {
    let data = &mut f as *mut _ as *mut libc::c_void;
    // SAFETY: trampoline correctly interprets data as our closure
    unsafe { set_handler(trampoline, data); }
    std::mem::forget(f); // Prevent drop — C now owns the lifetime
}
```

The trampoline function acts as a bridge between C's `void*` and Rust's typed closure.

### Lifetime Management

The closure must outlive the C callback registration. Common approaches include boxing and leaking, or using `Arc`:

```rust
use std::sync::Arc;

fn register_with_arc<F: Fn(i32) + Send + 'static>(callback: F) {
    let arc = Arc::new(callback);
    let raw = Arc::into_raw(arc) as *mut libc::c_void;

    // SAFETY: raw is a valid Arc pointer
    unsafe { set_handler(trampoline_arc, raw); }
}

extern "C" fn trampoline_arc(data: *mut libc::c_void, value: i32) {
    // SAFETY: data is a valid Arc pointer, we clone to avoid consuming it
    let arc = unsafe { Arc::from_raw(data as *const dyn Fn(i32)) };
    arc(value);
    // Don't drop — leak the Arc so it stays valid for future calls
    std::mem::forget(arc);
}
```

This is more robust because `Arc` handles reference counting.

## Common Pitfalls
1. **Dropping the closure while C still holds the pointer** — The closure must outlive the callback registration. Use `mem::forget` or `Arc` to prevent premature drops.
2. **Panicking in a callback** — Use `catch_unwind` in the trampoline to prevent panics from crossing into C.
3. **Capturing non-Send types in a callback used from multiple threads** — If C calls your callback from different threads, the closure must be `Send + Sync`.

## Best Practices
1. **Always use a trampoline for closures** — C cannot call Rust closures directly because closures carry environment data.
2. **Use `catch_unwind` in every trampoline** — Panics must never cross the FFI boundary.
3. **Document lifetime requirements** — Make it clear how long the callback must remain valid and how to unregister it.

## Summary
- Simple `extern "C" fn` functions can be passed directly as C callbacks.
- Closures require a trampoline function and a `void*` user data pointer.
- The closure must outlive the C callback registration — use `mem::forget` or `Arc`.
- Always use `catch_unwind` in trampoline functions to prevent panics from crossing FFI.
- Document lifetime and thread-safety requirements clearly.

## Code Examples

**Panic-safe callback trampoline pattern — passes Rust state to C via user data pointer with catch_unwind protection**

```rust
use std::panic;

unsafe extern "C" {
    fn c_register_callback(
        cb: extern "C" fn(*mut libc::c_void, i32) -> i32,
        user_data: *mut libc::c_void,
    );
}

struct CallbackState {
    name: String,
    count: usize,
}

// Trampoline with panic safety
extern "C" fn safe_trampoline(data: *mut libc::c_void, value: i32) -> i32 {
    let result = panic::catch_unwind(|| {
        // SAFETY: data points to a valid CallbackState
        let state = unsafe { &mut *(data as *mut CallbackState) };
        state.count += 1;
        println!("{}: received {} (call #{})", state.name, value, state.count);
        0 // success
    });
    result.unwrap_or(-1) // return error code on panic
}

fn register(name: &str) -> *mut CallbackState {
    let state = Box::new(CallbackState {
        name: name.to_string(),
        count: 0,
    });
    let raw = Box::into_raw(state);

    // SAFETY: raw is a valid pointer, trampoline handles it correctly
    unsafe { c_register_callback(safe_trampoline, raw as *mut libc::c_void); }
    raw // Return so caller can free later
}
```


## Resources

- [FFI — Callbacks](https://doc.rust-lang.org/nomicon/ffi.html#callbacks-from-c-code-to-rust-functions) — The Rustonomicon section on handling C callbacks in Rust

---

> 📘 *This lesson is part of the [Unsafe Rust & FFI](https://stanza.dev/courses/rust-unsafe) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*