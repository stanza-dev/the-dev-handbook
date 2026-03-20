---
source_course: "rust-unsafe"
source_lesson: "rust-unsafe-raw-pointers"
---

# Raw Pointers & Pointer Arithmetic

## Introduction
Raw pointers (`*const T` and `*mut T`) are the foundation of all unsafe memory access in Rust. Unlike references, they can be null, dangling, or unaligned. Understanding how to create, manipulate, and safely dereference raw pointers is essential for FFI, custom allocators, and high-performance data structures.

## Key Concepts
- **`*const T`**: An immutable raw pointer to a value of type `T`.
- **`*mut T`**: A mutable raw pointer that allows writing through the pointer.
- **Pointer arithmetic**: Moving a pointer forward or backward by a number of elements using `.add()`, `.sub()`, or `.offset()`.
- **Provenance**: The abstract concept of which allocation a pointer is "allowed" to access.

## Real World Context
Raw pointers are everywhere in systems programming. The Rust standard library uses them internally for `Vec`, `HashMap`, and `String`. If you work on embedded systems, write allocators, or integrate with C libraries, you will use raw pointers daily.

## Deep Dive

### Creating Raw Pointers

Creating a raw pointer is always safe — only dereferencing requires `unsafe`:

```rust
let x = 42;

// From references (safe — pointer is guaranteed valid at creation)
let ptr: *const i32 = &x;
let mut_ptr: *mut i32 = &x as *const i32 as *mut i32;

// Null pointers
let null: *const i32 = std::ptr::null();
let null_mut: *mut i32 = std::ptr::null_mut();
```

Note that creating a raw pointer from a reference is safe because the pointer inherits the validity of the reference. The danger comes later, when you dereference.

### Pointer Arithmetic

Rust provides safe arithmetic methods that operate in units of `T` (not bytes):

```rust
let arr = [10, 20, 30, 40, 50];
let ptr = arr.as_ptr();

unsafe {
    // .add(n) moves forward by n elements
    let third = *ptr.add(2);  // arr[2] = 30

    // .sub(n) moves backward by n elements
    let second = *ptr.add(2).sub(1);  // arr[1] = 20

    // .offset(n) can go forward (positive) or backward (negative)
    let fourth = *ptr.offset(3);  // arr[3] = 40
}
```

All pointer arithmetic is in units of `size_of::<T>()` bytes. For byte-level offsets, cast to `*const u8` first.

### Raw Pointer vs Reference Comparison

| Feature | `&T` / `&mut T` | `*const T` / `*mut T` |
|---------|------------------|------------------------|
| Null | Never | Can be null |
| Alignment | Always aligned | May be unaligned |
| Validity | Always valid | May be dangling |
| Aliasing | Enforced by borrow checker | Unchecked |
| Dereferencing | Safe | Requires `unsafe` |

### Null Checking Before Dereference

A common pattern is to check for null before dereferencing:

```rust
fn safe_deref(ptr: *const i32) -> Option<i32> {
    if ptr.is_null() {
        return None;
    }
    // SAFETY: We verified the pointer is not null.
    // Caller must still ensure it's valid and aligned.
    unsafe { Some(*ptr) }
}
```

Note that `is_null()` only checks for null — the pointer could still be dangling or unaligned.

## Common Pitfalls
1. **Assuming non-null means valid** — A pointer can be non-null but still dangling (pointing to freed memory) or unaligned. Always document the full set of preconditions.
2. **Byte vs element arithmetic** — `ptr.add(1)` advances by `size_of::<T>()` bytes, not 1 byte. Mixing this up causes out-of-bounds reads.
3. **Forgetting provenance** — Casting an integer to a pointer (`0xDEADBEEF as *const i32`) creates a pointer with no provenance, which is instant undefined behavior if dereferenced.

## Best Practices
1. **Prefer `NonNull<T>` over raw pointers** — When you know a pointer is never null, use `NonNull<T>` to encode that invariant in the type system.
2. **Use `slice::from_raw_parts` for contiguous data** — Rather than doing manual pointer arithmetic, convert to a slice and use safe indexing.
3. **Always document SAFETY comments** — Every `unsafe` dereference should have a comment explaining why the pointer is valid.

## Summary
- Raw pointers (`*const T`, `*mut T`) can be null, dangling, or unaligned.
- Creating raw pointers is safe; dereferencing them requires `unsafe`.
- Pointer arithmetic with `.add()` and `.sub()` operates in element-sized units, not bytes.
- Always check validity before dereferencing, and prefer `NonNull<T>` when null is impossible.
- Document safety invariants with `// SAFETY:` comments on every dereference.

## Code Examples

**Implementing split_at_mut using raw pointers — the borrow checker cannot prove two &mut slices from the same array don't overlap, so raw pointers are needed**

```rust
// Building a slice from raw parts and implementing split_at_mut
fn split_at_mut<T>(slice: &mut [T], mid: usize) -> (&mut [T], &mut [T]) {
    let len = slice.len();
    let ptr = slice.as_mut_ptr();

    assert!(mid <= len);

    // SAFETY: We verified mid <= len, so both sub-slices are within bounds.
    // The two slices do not overlap, so no aliasing &mut occurs.
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


## Resources

- [Raw Pointers — The Rustonomicon](https://doc.rust-lang.org/nomicon/meet-safe-and-unsafe.html) — Detailed guide to working with raw pointers in Rust

---

> 📘 *This lesson is part of the [Unsafe Rust & FFI](https://stanza.dev/courses/rust-unsafe) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*