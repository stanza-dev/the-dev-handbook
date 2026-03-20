---
source_course: "rust-unsafe"
source_lesson: "rust-unsafe-pin-self-referential"
---

# Pin & Self-Referential Structs

## Introduction
Self-referential structs — structs that contain pointers to their own fields — are fundamentally at odds with Rust's move semantics. Moving a value changes its memory address, which invalidates internal pointers. `Pin<P>` solves this by guaranteeing that a value will not be moved, making self-referential patterns safe.

## Key Concepts
- **`Pin<P>`**: A wrapper that guarantees the pointed-to value will not be moved from its current memory location.
- **`Unpin`**: A marker trait indicating a type can be safely moved even when pinned. Most types implement `Unpin` by default.
- **Self-referential struct**: A struct containing a pointer to one of its own fields. Moving the struct invalidates the pointer.
- **`pin!` macro**: Creates a pinned value on the stack (stable since Rust 1.68).

## Real World Context
Rust's `async`/`await` generates self-referential structs (futures that hold references across `.await` points). Every `async fn` relies on `Pin` to ensure the future is not moved while references into it are live. Libraries like `tokio` and `futures` use Pin extensively.

## Deep Dive

### The Problem: Moves Invalidate Internal Pointers

```rust
struct SelfRef {
    value: String,
    ptr: *const String, // Points to self.value
}

let mut s = SelfRef {
    value: String::from("hello"),
    ptr: std::ptr::null(),
};
s.ptr = &s.value; // ptr now points to s.value

// If s is moved, ptr still points to the OLD location!
let s2 = s; // MOVE! s.ptr is now dangling!
```

After the move, `s2.ptr` points to the memory where `s.value` used to be. Dereferencing it is UB.

### Pin Prevents Moves

```rust
use std::pin::Pin;
use std::marker::PhantomPinned;

struct PinnedSelfRef {
    value: String,
    ptr: *const String,
    _pin: PhantomPinned, // Opts out of Unpin
}

impl PinnedSelfRef {
    fn new(val: &str) -> Pin<Box<Self>> {
        let mut boxed = Box::new(PinnedSelfRef {
            value: val.to_string(),
            ptr: std::ptr::null(),
            _pin: PhantomPinned,
        });
        let self_ptr: *const String = &boxed.value;
        // SAFETY: We don't move the boxed value after setting the pointer
        unsafe {
            let mut_ref = Pin::as_mut(&mut Pin::new_unchecked(&mut *boxed));
            // Actually, we need to set it before pinning:
        }
        boxed.ptr = self_ptr;
        // SAFETY: value will not be moved because it's in a Box
        unsafe { Pin::new_unchecked(boxed) }
    }

    fn get_value(self: Pin<&Self>) -> &str {
        &self.value
    }

    fn get_ptr_value(self: Pin<&Self>) -> &str {
        // SAFETY: ptr was set to point to self.value and we're pinned
        unsafe { &*self.ptr }
    }
}
```

Once pinned, the value cannot be moved out. Any attempt to call `std::mem::swap` or move the value will fail to compile.

### The `pin!` Macro for Stack Pinning

```rust
use std::pin::pin;

let pinned = pin!(String::from("hello"));
// pinned: Pin<&mut String>
// The String is pinned to the stack frame
```

This is simpler than `Box::pin` for values that don't need to outlive the current scope.

### Pin and Async/Await

Every async function returns a type that implements `Future`. Futures that hold references across `.await` points are self-referential:

```rust
async fn example() {
    let data = vec![1, 2, 3];
    let reference = &data; // reference borrows data
    some_async_operation().await; // Suspend point!
    println!("{:?}", reference); // reference must still be valid
}
```

The generated future struct holds both `data` and `reference`. If the future were moved between the `.await` and the `println!`, `reference` would dangle. `Pin` prevents this move.

## Common Pitfalls
1. **Forgetting `PhantomPinned`** — Without it, your type implements `Unpin`, and Pin provides no protection. `PhantomPinned` opts out of `Unpin`.
2. **Moving after `Pin::new_unchecked`** — If you pin a value and then move it anyway (through unsafe code), you violate the Pin contract and cause UB.
3. **Confusing `Pin<Box<T>>` with `Box<Pin<T>>`** — `Pin<Box<T>>` pins the value inside the Box. `Box<Pin<T>>` boxes a Pin, which is rarely what you want.

## Best Practices
1. **Use `Box::pin()` for heap-pinned values** — It's the simplest way to create a `Pin<Box<T>>`.
2. **Use the `pin!` macro for stack pinning** — Avoids heap allocation when the pinned value doesn't need to outlive the scope.
3. **Implement `Unpin` only if your type has no self-references** — Most types are `Unpin` by default. Only opt out (with `PhantomPinned`) if you actually need pin guarantees.

## Summary
- Self-referential structs require `Pin` to prevent moves that would invalidate internal pointers.
- `Pin<P>` guarantees the pointed-to value will not be moved.
- `PhantomPinned` opts a type out of `Unpin`, making Pin actually protect against moves.
- Async/await relies on Pin because futures can be self-referential.
- Use `Box::pin()` for heap pinning and `pin!` for stack pinning.

## Code Examples

**A self-referential struct using Pin to prevent moves — the internal pointer remains valid because Pin guarantees the struct will not be relocated**

```rust
use std::pin::Pin;
use std::marker::PhantomPinned;

/// A self-referential struct where `ptr` points to `data`.
struct SelfReferential {
    data: String,
    ptr: *const String,
    _pin: PhantomPinned, // !Unpin: cannot be moved once pinned
}

impl SelfReferential {
    /// Create a new pinned instance on the heap.
    fn new(data: &str) -> Pin<Box<Self>> {
        let mut s = Box::new(SelfReferential {
            data: data.to_string(),
            ptr: std::ptr::null(),
            _pin: PhantomPinned,
        });
        // Set the self-referential pointer BEFORE pinning
        s.ptr = &s.data as *const String;

        // SAFETY: We will not move the value after this point
        // because it's behind Pin<Box<_>>
        unsafe { Pin::new_unchecked(s) }
    }

    /// Access the data through the self-referential pointer.
    fn get_via_ptr(self: Pin<&Self>) -> &str {
        // SAFETY: ptr is valid because self is pinned (cannot move)
        unsafe { &*self.ptr }
    }
}

let pinned = SelfReferential::new("hello");
assert_eq!(pinned.as_ref().get_via_ptr(), "hello");
```


## Resources

- [Pin — std documentation](https://doc.rust-lang.org/std/pin/index.html) — Comprehensive guide to Pin, Unpin, and pinning in Rust

---

> 📘 *This lesson is part of the [Unsafe Rust & FFI](https://stanza.dev/courses/rust-unsafe) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*