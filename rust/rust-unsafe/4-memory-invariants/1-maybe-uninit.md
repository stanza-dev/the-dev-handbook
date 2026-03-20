---
source_course: "rust-unsafe"
source_lesson: "rust-unsafe-maybe-uninit"
---

# MaybeUninit<T>

## Introduction
Uninitialized memory is one of the most dangerous areas of unsafe Rust. Reading an uninitialized value is always undefined behavior — even for types like `i32` that have no invalid bit patterns. `MaybeUninit<T>` is the safe way to work with memory that might not yet contain a valid value.

## Key Concepts
- **`MaybeUninit<T>`**: A union type that tells the compiler "this memory may or may not contain a valid `T`." It suppresses the validity requirement until you call `assume_init()`.
- **`assume_init()`**: Asserts that the `MaybeUninit` has been initialized and extracts the value. Calling this on uninitialized memory is UB.
- **`write()`**: Writes a value into the `MaybeUninit`, initializing it.
- **Validity invariant**: The rule that every value of a type must be one of the type's valid bit patterns.

## Real World Context
The standard library uses `MaybeUninit` internally for `Vec`, `HashMap`, and `BTreeMap` to initialize arrays element by element. Performance-sensitive code uses it to avoid default-initializing large buffers. FFI code uses it for C-style "out parameters."

## Deep Dive

### The Problem: Uninitialized Memory is Always UB

Even for simple types, reading uninitialized memory is undefined behavior:

```rust
// UB! Even though i32 has no invalid bit patterns
let x: i32;
println!("{x}"); // Compiler error in safe code, UB if you bypass it

// UB! The deprecated way
let x: bool = unsafe { std::mem::uninitialized() }; // NEVER DO THIS
// A bool must be 0 or 1; random bits could be 0x42, which is invalid
```

The old `std::mem::uninitialized()` function is deprecated because it creates instant UB for many types.

### MaybeUninit: The Correct Approach

MaybeUninit lets you safely represent uninitialized memory:

```rust
use std::mem::MaybeUninit;

// Create uninitialized storage
let mut slot: MaybeUninit<i32> = MaybeUninit::uninit();

// Write a value (initializes the memory)
slot.write(42);

// Now it's safe to read
let value = unsafe { slot.assume_init() };
assert_eq!(value, 42);
```

The key insight: `MaybeUninit::uninit()` is safe because it does not claim the memory contains a valid value. Only `assume_init()` requires `unsafe`, because that's where you assert initialization has happened.

### Initializing Arrays Element by Element

A common use case is building a large array without default values:

```rust
use std::mem::MaybeUninit;

fn create_squares() -> [i32; 100] {
    let mut arr: [MaybeUninit<i32>; 100] =
        [const { MaybeUninit::uninit() }; 100];

    for i in 0..100 {
        arr[i].write((i * i) as i32);
    }

    // SAFETY: All 100 elements were initialized in the loop
    unsafe {
        let ptr = arr.as_ptr() as *const [i32; 100];
        std::ptr::read(ptr)
    }
}
```

This avoids initializing the array to zero first and then overwriting every element.

### Out Parameter Pattern for FFI

C APIs often use out parameters. MaybeUninit is perfect for this:

```rust
unsafe extern "C" {
    fn get_system_info(out: *mut SystemInfo) -> libc::c_int;
}

fn fetch_info() -> Option<SystemInfo> {
    let mut info = MaybeUninit::<SystemInfo>::uninit();
    // SAFETY: get_system_info writes to the pointer on success
    let result = unsafe { get_system_info(info.as_mut_ptr()) };
    if result == 0 {
        // SAFETY: result == 0 means the C function initialized the struct
        Some(unsafe { info.assume_init() })
    } else {
        None
    }
}
```

This avoids constructing a dummy value just to have it overwritten.

### Never Use `mem::zeroed()` Carelessly

Zeroing memory is only safe for types where all-zeros is valid:

```rust
// SAFE: all-zeros is a valid i32
let n: i32 = unsafe { std::mem::zeroed() };

// UB! String expects a valid heap pointer, not null
let s: String = unsafe { std::mem::zeroed() };

// UB! References can never be null
let r: &i32 = unsafe { std::mem::zeroed() };
```

When in doubt, use `MaybeUninit` instead of `zeroed()`.

## Common Pitfalls
1. **Calling `assume_init()` before initialization** — This is UB, even for integer types. The compiler may optimize based on the assumption that initialized values are valid.
2. **Partial initialization of arrays** — If you initialize only some elements and then call `assume_init()`, the uninitialized elements are UB. Use a counter to track how many elements have been initialized.
3. **Using `mem::zeroed()` for types with validity invariants** — References, `NonNull`, `bool`, and `String` all have bit patterns that zero does not satisfy.

## Best Practices
1. **Use `MaybeUninit` instead of `mem::uninitialized`** — The old function is deprecated and almost always UB. `MaybeUninit` is the correct replacement.
2. **Track initialization state** — When initializing arrays or buffers incrementally, use a length counter so you know which elements are initialized.
3. **Prefer `write()` over `as_mut_ptr()`** — `write()` is clearer about intent and handles dropping the previous value if needed.

## Summary
- `MaybeUninit<T>` safely represents potentially uninitialized memory.
- `assume_init()` is the unsafe assertion that initialization has occurred.
- Use `write()` to initialize a `MaybeUninit` slot.
- Never use `mem::uninitialized()` — it is deprecated and almost always UB.
- `mem::zeroed()` is only safe for types where all-zeros is a valid value.

## Code Examples

**Generic array initialization using MaybeUninit — avoids default construction by initializing each element individually with a closure**

```rust
use std::mem::MaybeUninit;

/// Initialize an array using a closure, element by element.
fn init_array<T, F, const N: usize>(mut f: F) -> [T; N]
where
    F: FnMut(usize) -> T,
{
    let mut arr: [MaybeUninit<T>; N] =
        [const { MaybeUninit::uninit() }; N];

    for (i, elem) in arr.iter_mut().enumerate() {
        elem.write(f(i));
    }

    // SAFETY: All N elements were initialized in the loop above
    unsafe {
        let ptr = arr.as_ptr() as *const [T; N];
        std::ptr::read(ptr)
    }
}

// Usage: create an array of squares
let squares: [i32; 10] = init_array(|i| (i * i) as i32);
assert_eq!(squares, [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]);

// Usage: create an array of formatted strings
let labels: [String; 5] = init_array(|i| format!("item_{i}"));
assert_eq!(labels[2], "item_2");
```


## Resources

- [MaybeUninit — std documentation](https://doc.rust-lang.org/std/mem/union.MaybeUninit.html) — Complete API reference for MaybeUninit including safety requirements

---

> 📘 *This lesson is part of the [Unsafe Rust & FFI](https://stanza.dev/courses/rust-unsafe) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*