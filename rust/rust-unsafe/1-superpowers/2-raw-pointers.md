---
source_course: "rust-unsafe"
source_lesson: "rust-unsafe-raw-pointers"
---

# Raw Pointers: *const T and *mut T

## Creating Raw Pointers

```rust
let x = 42;

// From references (safe)
let ptr: *const i32 = &x;
let mut_ptr: *mut i32 = &x as *const i32 as *mut i32;

// From integers (very unsafe!)
let addr = 0xDEADBEEF as *const i32;

// Null pointers
let null: *const i32 = std::ptr::null();
let null_mut: *mut i32 = std::ptr::null_mut();
```

## Pointer Operations

```rust
let arr = [1, 2, 3, 4, 5];
let ptr = arr.as_ptr();

unsafe {
    // Offset (in elements)
    let third = *ptr.add(2);  // arr[2] = 3
    
    // Negative offset
    let second = *ptr.add(2).sub(1);  // arr[1] = 2
    
    // Byte offset
    let byte_ptr = (ptr as *const u8).add(8);  // 8 bytes = 2 i32s
    
    // offset() can go negative
    let elem = *ptr.offset(3);  // arr[3] = 4
}
```

## Raw Pointer vs Reference

| Feature | &T / &mut T | *const T / *mut T |
|---------|-------------|-------------------|
| Null | Never | Can be null |
| Alignment | Always aligned | May be unaligned |
| Validity | Always valid | May be dangling |
| Aliasing | Enforced | Unchecked |
| Dereferencing | Safe | Unsafe |

## Common Patterns

```rust
// Checking validity before deref
fn safe_deref<T>(ptr: *const T) -> Option<&T> {
    if ptr.is_null() {
        return None;
    }
    // SAFETY: We checked for null
    unsafe { Some(&*ptr) }
}

// Pointer comparison
let a = [1, 2, 3];
let p1 = a.as_ptr();
let p2 = unsafe { p1.add(1) };
assert!(p1 < p2);  // Pointer ordering
```

## Code Examples

**Raw pointer slice operations**

```rust
// Building a simple slice from raw parts
fn slice_from_raw<T>(ptr: *const T, len: usize) -> &[T] {
    // SAFETY: Caller must ensure:
    // - ptr is valid for `len` elements
    // - Memory won't be mutated
    // - Lifetime is valid
    unsafe { std::slice::from_raw_parts(ptr, len) }
}

// Implementing split_at_mut using raw pointers
fn split_at_mut<T>(slice: &mut [T], mid: usize) -> (&mut [T], &mut [T]) {
    let len = slice.len();
    let ptr = slice.as_mut_ptr();
    
    assert!(mid <= len);
    
    unsafe {
        (
            std::slice::from_raw_parts_mut(ptr, mid),
            std::slice::from_raw_parts_mut(ptr.add(mid), len - mid),
        )
    }
}

let mut arr = [1, 2, 3, 4, 5];
let (left, right) = split_at_mut(&mut arr, 2);
left[0] = 10;
right[0] = 30;
assert_eq!(arr, [10, 2, 30, 4, 5]);
```


---

> 📘 *This lesson is part of the [Unsafe Rust & FFI](https://stanza.dev/courses/rust-unsafe) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*