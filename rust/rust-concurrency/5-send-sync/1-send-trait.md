---
source_course: "rust-concurrency"
source_lesson: "rust-concurrency-send-trait"
---

# The Send Trait

## Introduction

Rust's fearless concurrency guarantee rests on two marker traits: `Send` and `Sync`. The `Send` trait marks types whose ownership can be safely transferred to another thread. Without `Send`, the compiler refuses to let you move a value into a `thread::spawn` closure — this single rule prevents an entire class of data races at compile time.

## Key Concepts

- **Send**: A marker trait (no methods) that indicates a type is safe to move to another thread. If a type is `Send`, you can transfer ownership of a value of that type from one thread to another.
- **Auto-trait**: `Send` is automatically implemented for any type whose fields are all `Send`. You never need to write `impl Send for MyStruct` unless you are wrapping a non-`Send` raw pointer and can guarantee safety.
- **!Send types**: Types that are explicitly *not* `Send`. The most common are `Rc<T>` (non-atomic reference count), raw pointers (`*const T`, `*mut T`), and `MutexGuard<T>` (must be dropped on the same thread that locked it).
- **thread::spawn bounds**: The closure passed to `thread::spawn` must be `Send + 'static`, and its return type must also be `Send + 'static`. This is the compiler's enforcement point — if any captured variable is not `Send`, the call fails to compile.

## Real World Context

Every time you call `thread::spawn`, the compiler checks that every value moved into the closure is `Send`. This prevents you from accidentally sharing an `Rc` across threads (which would corrupt its reference count) or sending a `MutexGuard` to another thread (which would violate the lock's invariants). In practice, most types you write are automatically `Send` because they are composed of `Send` fields. The compiler only stops you when you reach for something inherently thread-unsafe.

## Deep Dive

`Send` is defined as a marker trait with no methods. The compiler auto-implements it for types composed entirely of `Send` fields.

```rust
// Send is defined roughly as:
pub unsafe auto trait Send {}
```

Because `Send` is an auto-trait, your own structs get it for free when all their fields are `Send`.

```rust
use std::sync::Arc;
use std::rc::Rc;

struct MySendType {
    name: String,       // Send
    count: Arc<i32>,    // Send
    values: Vec<u64>,   // Send
}
// MySendType is automatically Send because all fields are Send.

struct NotSend {
    data: Rc<String>,   // Rc is !Send
}
// NotSend is NOT Send because Rc<String> is !Send.
```

The `thread::spawn` function enforces `Send` at compile time. Both the closure and its return type must be `Send`.

```rust
use std::thread;

// This compiles — String and usize are both Send.
let s = String::from("hello");
let handle = thread::spawn(move || {
    println!("{s}");
    s.len() // usize is Send
});
let length: usize = handle.join().unwrap();
```

Attempting to send an `Rc` to another thread produces a compile error.

```rust
use std::rc::Rc;
use std::thread;

let rc = Rc::new(42);
// This fails to compile:
// thread::spawn(move || {
//     println!("{}", *rc);
// });
// Error: `Rc<i32>` cannot be sent between threads safely
```

The reason `Rc` is `!Send` is that its reference count uses non-atomic operations. If two threads could hold clones of the same `Rc`, they could increment or decrement the count simultaneously, causing a data race.

```rust
use std::rc::Rc;

let rc1 = Rc::new(5);
let rc2 = Rc::clone(&rc1); // non-atomic refcount increment

// If rc1 were sent to another thread:
// Thread A: drop(rc1) — decrements count non-atomically
// Thread B: drop(rc2) — decrements count non-atomically
// Both decrements happen simultaneously — data race on the count!
// The fix: use Arc, which uses atomic operations for the count.
```

Raw pointers are `!Send` by default because the compiler cannot verify that the pointed-to data is safe to access from another thread.

```rust
struct Wrapper {
    ptr: *mut u8,
}
// Wrapper is !Send because *mut u8 is !Send.

// If you can guarantee thread safety, you can opt in manually:
unsafe impl Send for Wrapper {}
// This is YOUR responsibility — the compiler trusts you.
```

You can verify whether a type is `Send` at compile time with a helper function.

```rust
fn assert_send<T: Send>() {}

assert_send::<String>();       // compiles
assert_send::<Vec<i32>>();     // compiles
assert_send::<std::sync::Arc<String>>(); // compiles
// assert_send::<Rc<i32>>();   // compile error: Rc is !Send
// assert_send::<*mut u8>();   // compile error: raw pointers are !Send
```

## Common Pitfalls

1. **Wrapping a non-Send type does not make it Send** — If your struct contains an `Rc` or a raw pointer, the entire struct becomes `!Send`. You must replace the non-Send field (e.g., use `Arc` instead of `Rc`) or use `unsafe impl Send` with a correctness proof.
2. **Confusing Send with Copy or Clone** — `Send` is about thread safety, not about duplication. A `String` is `Send` (safe to move to another thread) but not `Copy`. An `Rc` is `Clone` but not `Send`.

## Best Practices

1. **Let the compiler derive Send automatically** — In almost all cases, your types will be `Send` because their fields are `Send`. Only reach for `unsafe impl Send` when wrapping FFI pointers or other raw types where you can guarantee safety.
2. **Use `Arc` instead of `Rc` for shared ownership across threads** — `Arc` uses atomic reference counting and is `Send + Sync`. It is the direct thread-safe replacement for `Rc`.

## Summary

- `Send` means a value can be safely moved to another thread.
- It is an auto-trait: your type is `Send` if all its fields are `Send`.
- `Rc`, raw pointers, and `MutexGuard` are `!Send`.
- `thread::spawn` enforces `Send + 'static` on the closure and its return type.
- Use `unsafe impl Send` only when you can prove thread safety for a raw-pointer-containing type.

## Code Examples

**Compile-time Send verification and thread::spawn ownership transfer**

```rust
use std::thread;
use std::sync::Arc;

// Compile-time Send check helper
fn assert_send<T: Send>() {}

fn main() {
    // These all compile — all are Send
    assert_send::<String>();
    assert_send::<Vec<i32>>();
    assert_send::<Arc<String>>();
    assert_send::<i32>();

    // Rc is NOT Send — this would fail:
    // assert_send::<std::rc::Rc<i32>>();

    // thread::spawn enforces Send on the closure and return type
    let data = vec![1, 2, 3];
    let handle = thread::spawn(move || {
        let sum: i32 = data.iter().sum();
        println!("sum = {sum}");
        sum // return type must also be Send
    });

    let result = handle.join().unwrap();
    println!("thread returned: {result}");
}
```


## Resources

- [Send and Sync — The Rustonomicon](https://doc.rust-lang.org/nomicon/send-and-sync.html) — Unsafe Rust guide covering Send and Sync marker traits
- [std::marker::Send](https://doc.rust-lang.org/std/marker/trait.Send.html) — Standard library reference for the Send trait
- [Fearless Concurrency — The Rust Book](https://doc.rust-lang.org/book/ch16-00-concurrency.html) — Official Rust Book chapter on concurrency and thread safety

---

> 📘 *This lesson is part of the [Concurrent Rust](https://stanza.dev/courses/rust-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*