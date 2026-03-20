---
source_course: "rust-unsafe"
source_lesson: "rust-unsafe-safety-contracts"
---

# Safety Contracts & Sound Abstractions

## Introduction
Writing `unsafe` code is only half the challenge. The other half is proving to yourself (and future maintainers) that the code is correct. Rust's ecosystem has developed conventions for documenting safety requirements, and the concept of "soundness" is central to building reliable unsafe abstractions.

## Key Concepts
- **Soundness**: A safe API is sound if no sequence of safe calls can trigger undefined behavior. An unsound API has a bug.
- **Safety contract**: The set of preconditions a caller must satisfy before calling an `unsafe fn`.
- **SAFETY comment**: A `// SAFETY:` comment documenting why a specific `unsafe` block upholds its invariants.
- **`# Safety` doc section**: The `///` doc comment section on an `unsafe fn` that lists the caller's obligations.

## Real World Context
Every crate that exposes unsafe APIs — from `std` to `libc` to `tokio` — uses these conventions. Clippy's `undocumented_unsafe_blocks` lint enforces `// SAFETY:` comments. When reviewing PRs that touch unsafe code, these comments are what reviewers focus on.

## Deep Dive

### The `# Safety` Section

Every `unsafe fn` must have a `# Safety` section in its doc comment listing the caller's obligations:

```rust
/// Reads a value from the given pointer.
///
/// # Safety
/// - `ptr` must be non-null
/// - `ptr` must be properly aligned for `T`
/// - `ptr` must point to a valid, initialized `T`
/// - The memory must not be concurrently mutated
unsafe fn read_value<T>(ptr: *const T) -> T {
    unsafe { std::ptr::read(ptr) }
}
```

This is not just a convention — it is the contract that makes the function usable.

### The `// SAFETY:` Comment

Every `unsafe` block should have a comment directly above it explaining why it is sound:

```rust
fn get_first(slice: &[i32]) -> Option<i32> {
    if slice.is_empty() {
        return None;
    }
    let ptr = slice.as_ptr();
    // SAFETY: We checked the slice is non-empty, so ptr points
    // to at least one valid i32. The slice borrow keeps it alive.
    Some(unsafe { *ptr })
}
```

The comment should reference the specific preconditions being satisfied, not just say "this is safe."

### Building Sound Abstractions

The pattern for sound unsafe code is: unsafe internals behind a safe public API.

```rust
pub struct FixedBuffer {
    data: *mut u8,
    len: usize,
    cap: usize,
}

impl FixedBuffer {
    pub fn new(capacity: usize) -> Self {
        let layout = std::alloc::Layout::array::<u8>(capacity).unwrap();
        // SAFETY: layout has non-zero size (capacity > 0 checked by Layout)
        let data = unsafe { std::alloc::alloc(layout) };
        if data.is_null() {
            std::alloc::handle_alloc_error(layout);
        }
        FixedBuffer { data, len: 0, cap: capacity }
    }

    pub fn push(&mut self, byte: u8) -> bool {
        if self.len >= self.cap {
            return false;
        }
        // SAFETY: len < cap, so data.add(len) is within the allocation
        unsafe { self.data.add(self.len).write(byte); }
        self.len += 1;
        true
    }
}
```

Because `push` maintains the invariant that `len <= cap`, callers never need `unsafe`.

## Common Pitfalls
1. **Writing `// SAFETY: this is safe`** — This is useless. The comment must explain *why* it is safe by referencing specific invariants.
2. **Exposing unsound safe APIs** — If your safe wrapper can be made to trigger UB through any sequence of safe calls, the abstraction is unsound. Test edge cases rigorously.
3. **Forgetting to document all preconditions** — If an `unsafe fn` has five preconditions but you only document three, callers will violate the undocumented ones.

## Best Practices
1. **Enable `clippy::undocumented_unsafe_blocks`** — This lint ensures every `unsafe` block has a `// SAFETY:` comment.
2. **Test with Miri** — Run `cargo miri test` to detect undefined behavior in your unsafe code. Miri catches use-after-free, out-of-bounds access, and aliasing violations.
3. **Minimize the unsafe surface area** — The fewer `unsafe` blocks you have, the fewer places bugs can hide. Extract the unsafe operation into the smallest possible helper function.

## Summary
- Every `unsafe fn` needs a `# Safety` doc section listing caller obligations.
- Every `unsafe` block needs a `// SAFETY:` comment explaining why invariants hold.
- Sound abstractions wrap unsafe internals in safe public APIs.
- Use Miri (`cargo miri test`) to detect undefined behavior in tests.
- Enable `clippy::undocumented_unsafe_blocks` to enforce documentation.

## Code Examples

**A sound wrapper around NonNull that maintains validity invariants — the safe from_ref constructor guarantees correctness, while from_raw pushes the obligation to the caller**

```rust
/// A non-null, aligned pointer wrapper.
///
/// # Invariants
/// - `ptr` is always non-null
/// - `ptr` is always properly aligned for `T`
/// - `ptr` points to a valid, initialized `T`
pub struct ValidPtr<T> {
    ptr: std::ptr::NonNull<T>,
}

impl<T> ValidPtr<T> {
    /// Creates a new ValidPtr from a reference.
    /// This is always safe because references are always valid.
    pub fn from_ref(r: &T) -> Self {
        ValidPtr {
            ptr: std::ptr::NonNull::from(r),
        }
    }

    /// # Safety
    /// `ptr` must be non-null, aligned, and point to a valid `T`.
    pub unsafe fn from_raw(ptr: *mut T) -> Self {
        ValidPtr {
            // SAFETY: Caller guarantees ptr is non-null
            ptr: unsafe { std::ptr::NonNull::new_unchecked(ptr) },
        }
    }

    pub fn as_ref(&self) -> &T {
        // SAFETY: Our invariants guarantee the pointer is valid
        unsafe { self.ptr.as_ref() }
    }
}
```


## Resources

- [The Rustonomicon — Working with Unsafe](https://doc.rust-lang.org/nomicon/working-with-unsafe.html) — Guidelines for writing correct unsafe code and sound abstractions

---

> 📘 *This lesson is part of the [Unsafe Rust & FFI](https://stanza.dev/courses/rust-unsafe) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*