---
source_course: "rust-traits"
source_lesson: "rust-traits-auto-trait-rules"
---

# Auto Trait Implementation Rules

## Introduction
Auto traits like `Send`, `Sync`, and `Unpin` are implemented automatically by the compiler based on a type's fields. Understanding the rules for when auto traits are and are not implemented is essential for writing correct concurrent Rust code.

## Key Concepts
- **Auto Implementation**: The compiler implements auto traits for a type if and only if all of its fields implement that trait.
- **Negative Reasoning**: A type does NOT implement an auto trait if any field fails to implement it.
- **Manual Override**: You can manually implement auto traits with `unsafe impl` or opt out with negative impls (nightly) or `PhantomData` tricks (stable).

## Real World Context
When you get a compiler error saying a type is not `Send` or `Sync`, it is because one of its fields (or a field of a field) does not implement that trait. Common culprits include `Rc<T>` (not `Send`), `Cell<T>` (not `Sync`), and raw pointers (neither `Send` nor `Sync`).

## Deep Dive

### How Auto Traits Propagate

```rust
// All fields are Send, so MyStruct is Send
struct MyStruct {
    name: String,    // Send ✓
    count: i32,      // Send ✓
    data: Vec<u8>,   // Send ✓
}
// MyStruct: Send ✓ (auto-implemented)

// One field is not Send, so NotSendStruct is not Send
use std::rc::Rc;
struct NotSendStruct {
    name: String,    // Send ✓
    shared: Rc<i32>, // Send ✗ (Rc is not Send)
}
// NotSendStruct: Send ✗
```

### Common Non-Send/Non-Sync Types

| Type | Send | Sync | Why |
|------|------|------|-----|
| `Rc<T>` | No | No | Reference counting is not atomic |
| `Cell<T>` | Yes | No | Interior mutability without synchronization |
| `RefCell<T>` | Yes | No | Runtime borrow checking is not thread-safe |
| `*const T` | No | No | Raw pointers have no safety guarantees |
| `Arc<T>` | Yes* | Yes* | *When T: Send + Sync |
| `Mutex<T>` | Yes* | Yes | *When T: Send |

### Opting Out on Stable Rust

Since negative impls (`impl !Send`) are nightly-only, use `PhantomData` to opt out:

```rust
use std::marker::PhantomData;

struct NotSend {
    data: i32,
    _not_send: PhantomData<*const ()>, // *const () is not Send
}
// NotSend is now !Send and !Sync
```

### Unsafe Manual Implementation

When wrapping raw pointers in a type you know is safe to send:

```rust
struct SafeWrapper {
    ptr: *mut u8,
}

// SAFETY: We ensure the pointer is only accessed from one thread at a time
unsafe impl Send for SafeWrapper {}
unsafe impl Sync for SafeWrapper {}
```

## Common Pitfalls
1. **Accidentally breaking Send/Sync by adding a field** — Adding an `Rc<T>` or `Cell<T>` field to a struct removes `Send` or `Sync` for the entire type.
2. **Incorrectly implementing Send/Sync with unsafe** — Manually implementing `Send` or `Sync` for a type that is not actually thread-safe causes undefined behavior.

## Best Practices
1. **Let the compiler derive auto traits** — Only add manual `unsafe impl Send` when wrapping FFI types or raw pointers where you can prove safety.
2. **Use `Arc<Mutex<T>>` instead of raw pointers** — Safe abstractions are almost always preferable to manual `unsafe impl`.
3. **Test with static assertions** — Use `fn assert_send<T: Send>() {}` followed by `assert_send::<MyType>()` to verify your types at compile time.

## Summary
- Auto traits are implemented when all fields implement them.
- `Rc`, `Cell`, `RefCell`, and raw pointers are common sources of non-Send/non-Sync types.
- Use `PhantomData<*const ()>` to opt out of Send/Sync on stable Rust.
- Manual `unsafe impl Send/Sync` requires a proof of safety.

## Code Examples

**Static assertions verify at compile time whether a type implements Send and Sync, catching thread-safety issues immediately**

```rust
use std::marker::PhantomData;

// Compile-time assertion that a type implements Send
fn assert_send<T: Send>() {}
fn assert_sync<T: Sync>() {}

struct ThreadSafe {
    data: String,
    count: i32,
}

// These compile: ThreadSafe is Send + Sync
assert_send::<ThreadSafe>();
assert_sync::<ThreadSafe>();

struct NotThreadSafe {
    data: String,
    _no_send: PhantomData<*const ()>,
}

// assert_send::<NotThreadSafe>(); // Compile error!
// assert_sync::<NotThreadSafe>(); // Compile error!
```


## Resources

- [Send and Sync](https://doc.rust-lang.org/nomicon/send-and-sync.html) — Rustonomicon chapter on Send, Sync, and thread safety

---

> 📘 *This lesson is part of the [Advanced Traits & Generics](https://stanza.dev/courses/rust-traits) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*