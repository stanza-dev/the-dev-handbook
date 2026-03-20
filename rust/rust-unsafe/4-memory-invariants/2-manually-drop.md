---
source_course: "rust-unsafe"
source_lesson: "rust-unsafe-manually-drop"
---

# ManuallyDrop<T>

## Introduction
Rust automatically runs destructors when values go out of scope. `ManuallyDrop<T>` gives you explicit control over when (or whether) the destructor runs. This is essential for FFI ownership transfer, union types with drop fields, and custom drop implementations.

## Key Concepts
- **`ManuallyDrop<T>`**: A wrapper that prevents automatic destructor execution. The inner value is not dropped when the wrapper goes out of scope.
- **`ManuallyDrop::drop()`**: Manually runs the destructor. After this call, the inner value is invalid.
- **`mem::forget()`**: Prevents a value's destructor from running by consuming it. Leaking memory is safe (not UB) but usually a bug.
- **`drop_in_place()`**: Runs the destructor for a value behind a raw pointer without deallocating memory.

## Real World Context
The standard library uses `ManuallyDrop` internally in `Vec`, `HashMap`, and `BTreeMap` to manage element destruction during resizing. FFI code uses it to transfer ownership to C without running Rust destructors. Custom collections use it with `ptr::drop_in_place` for fine-grained control.

## Deep Dive

### Basic Usage

Wrap a value to prevent automatic drop:

```rust
use std::mem::ManuallyDrop;

let mut x = ManuallyDrop::new(String::from("hello"));

// Use the value normally through Deref
println!("{}", *x); // prints "hello"

// Explicitly drop when ready
unsafe {
    ManuallyDrop::drop(&mut x);
}
// WARNING: do not use x after this point!
```

The `drop` call is unsafe because using the value afterward is UB.

### Use Case 1: Unions with Drop Types

Rust unions cannot contain types that implement `Drop` unless wrapped in `ManuallyDrop`:

```rust
union StringOrInt {
    s: ManuallyDrop<String>,
    n: i32,
}

let mut u = StringOrInt {
    s: ManuallyDrop::new(String::from("hello")),
};

// Must manually drop before switching variants
unsafe {
    ManuallyDrop::drop(&mut u.s);
    u.n = 42;
}
```

Without `ManuallyDrop`, the compiler wouldn't know which variant to drop.

### Use Case 2: FFI Ownership Transfer

Transfer a Rust value to C without running the destructor:

```rust
/// Give a Vec's buffer to C. C now owns the memory.
fn transfer_to_c(data: Vec<u8>) -> (*mut u8, usize, usize) {
    let mut data = ManuallyDrop::new(data);
    let ptr = data.as_mut_ptr();
    let len = data.len();
    let cap = data.capacity();
    (ptr, len, cap)
    // Vec's destructor does NOT run — C owns the memory now
}
```

The C side must eventually call back into Rust to free the memory.

### Use Case 3: Custom Drop with `drop_in_place`

For custom collections, you often need to drop elements individually:

```rust
struct MyVec<T> {
    ptr: *mut T,
    len: usize,
    cap: usize,
}

impl<T> Drop for MyVec<T> {
    fn drop(&mut self) {
        // Drop all elements individually
        for i in 0..self.len {
            // SAFETY: ptr.add(i) points to an initialized T for i < len
            unsafe {
                std::ptr::drop_in_place(self.ptr.add(i));
            }
        }
        // Then deallocate the buffer
        if self.cap > 0 {
            let layout = std::alloc::Layout::array::<T>(self.cap).unwrap();
            // SAFETY: ptr was allocated with this layout
            unsafe { std::alloc::dealloc(self.ptr as *mut u8, layout); }
        }
    }
}
```

This separates element destruction from buffer deallocation.

### `mem::forget`: The Safe Leak

`mem::forget` is safe because leaking memory is not undefined behavior:

```rust
let s = String::from("hello");
std::mem::forget(s); // String is leaked — memory is never freed
```

However, leaking is usually a bug. The main legitimate use is when another system (like C code) takes ownership.

## Common Pitfalls
1. **Using a value after `ManuallyDrop::drop`** — The value is invalid after dropping. Any access is UB.
2. **Forgetting to drop** — If you never call `ManuallyDrop::drop`, the inner value leaks. This is safe but wastes resources.
3. **Confusing `ManuallyDrop` with `MaybeUninit`** — `ManuallyDrop` contains a valid value that you choose not to drop. `MaybeUninit` may not contain a valid value at all.

## Best Practices
1. **Prefer `ManuallyDrop` over `mem::forget`** — `ManuallyDrop` makes the intent clearer and gives you the option to drop later.
2. **Use `take()` instead of `drop()` when possible** — `ManuallyDrop::take()` moves the inner value out, which is safer because you get a regular owned value.
3. **Document the drop responsibility** — When transferring ownership via ManuallyDrop, document who is responsible for eventual cleanup.

## Summary
- `ManuallyDrop<T>` prevents automatic destructor execution.
- Call `ManuallyDrop::drop()` to explicitly run the destructor (unsafe).
- Use it for: unions with Drop types, FFI ownership transfer, and custom drop logic.
- `mem::forget` is safe but usually indicates a bug.
- Never access a value after calling `ManuallyDrop::drop()` on it.

## Code Examples

**ManuallyDrop for FFI ownership transfer — gives a Vec's buffer to C without running the destructor, with a matching reclaim function**

```rust
use std::mem::ManuallyDrop;

/// Transfer a Vec<u8> buffer to C without running Rust's destructor.
/// Returns (pointer, length, capacity). Caller must call `reclaim_buffer`
/// to free the memory.
fn give_buffer_to_c(data: Vec<u8>) -> (*mut u8, usize, usize) {
    let mut md = ManuallyDrop::new(data);
    (md.as_mut_ptr(), md.len(), md.capacity())
}

/// Reclaim a buffer previously given to C.
/// # Safety
/// ptr, len, cap must be from a previous `give_buffer_to_c` call.
unsafe fn reclaim_buffer(ptr: *mut u8, len: usize, cap: usize) {
    // SAFETY: Reconstructing the Vec from its raw parts
    let _ = unsafe { Vec::from_raw_parts(ptr, len, cap) };
    // Vec is dropped here, freeing the memory
}

// Usage
let buffer = vec![1u8, 2, 3, 4, 5];
let (ptr, len, cap) = give_buffer_to_c(buffer);
// ... C uses the buffer ...
// When done:
unsafe { reclaim_buffer(ptr, len, cap); }
```


## Resources

- [ManuallyDrop — std documentation](https://doc.rust-lang.org/std/mem/struct.ManuallyDrop.html) — API reference for ManuallyDrop including usage patterns and safety

---

> 📘 *This lesson is part of the [Unsafe Rust & FFI](https://stanza.dev/courses/rust-unsafe) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*