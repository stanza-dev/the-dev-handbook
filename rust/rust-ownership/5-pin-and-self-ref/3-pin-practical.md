---
source_course: "rust-ownership"
source_lesson: "rust-ownership-pin-practical"
---

# Pin in Practice

## Introduction
Knowing what Pin is and why it exists is one thing — using it day-to-day is another. This lesson covers the practical patterns: creating pinned values, projecting through Pin to access fields, and using the pin-project crate to avoid unsafe code.

## Key Concepts
- **Box::pin**: Creates a `Pin<Box<T>>` by allocating on the heap. The simplest way to pin a value.
- **pin! macro**: Creates a pinned value on the stack (Rust 1.68+). Useful in async code to pin a future without heap allocation.
- **Pin projection**: Accessing a field of a pinned struct while preserving the Pin guarantee on that field.

## Real World Context
Every async runtime (Tokio, async-std) uses Pin internally to poll futures safely. When you write custom Stream or Future implementations, you need pin projection. The pin-project crate is used in virtually every production async Rust codebase to handle this safely.

## Deep Dive
The most common way to create pinned values is with `Box::pin`:

```rust
use std::pin::Pin;

struct MyType { data: String }

let pinned: Pin<Box<MyType>> = Box::pin(MyType {
    data: String::from("hello"),
});
```

For stack pinning without heap allocation, use the `pin!` macro:

```rust
use std::pin::pin;
use std::future::Future;
use std::task::{Context, Poll};

async fn do_work() -> i32 { 42 }

// Pin a future on the stack
let mut future = pin!(do_work());
// future is Pin<&mut impl Future<Output = i32>>
```

Pin projection — accessing fields of a pinned struct — requires care. Manually, it involves unsafe:

```rust
struct Wrapper {
    inner: InnerType,
    count: usize,
}

impl Wrapper {
    fn inner(self: Pin<&mut Self>) -> Pin<&mut InnerType> {
        // Only safe if InnerType is structurally pinned
        unsafe { self.map_unchecked_mut(|s| &mut s.inner) }
    }
    
    fn count(self: Pin<&mut Self>) -> &mut usize {
        // Safe: usize is Unpin, so no pinning needed
        unsafe { &mut self.get_unchecked_mut().count }
    }
}
```

The pin-project crate eliminates the unsafe entirely:

```rust
use pin_project::pin_project;
use std::future::Future;
use std::pin::Pin;
use std::task::{Context, Poll};

#[pin_project]
struct MyFuture<F: Future> {
    #[pin]  // This field is structurally pinned
    inner: F,
    count: usize,  // Not pinned — can be accessed as &mut
}

impl<F: Future> Future for MyFuture<F> {
    type Output = F::Output;
    
    fn poll(self: Pin<&mut Self>, cx: &mut Context) -> Poll<F::Output> {
        let this = self.project();
        // this.inner: Pin<&mut F>
        // this.count: &mut usize
        this.inner.poll(cx)
    }
}
```

## Common Pitfalls
1. **Writing unsafe pin projection manually** — It is easy to get wrong. Use pin-project instead, which is sound and well-tested.
2. **Heap-allocating when stack pinning suffices** — For short-lived pinned values (like polling a future in a loop), `pin!` avoids the heap allocation cost of `Box::pin`.

## Best Practices
1. **Use pin-project for all custom Future/Stream implementations** — It is the standard way to handle pin projection safely.
2. **Use Box::pin to erase complex future types** — `Pin<Box<dyn Future>>` simplifies type signatures when the concrete future type is complex or opaque.

## Summary
- Box::pin creates heap-pinned values; pin! creates stack-pinned values.
- Pin projection allows accessing fields of pinned structs while maintaining guarantees.
- The pin-project crate provides safe pin projection without unsafe code.
- Use pin-project for any custom Future or Stream implementation.

## Code Examples

**Stack-pinned future polling — a minimal block_on executor using pin! and a no-op waker**

```rust
use std::future::Future;
use std::pin::pin;
use std::task::{Context, Poll, Wake};
use std::sync::Arc;

async fn do_work() -> i32 {
    42
}

// A minimal block_on using stack pinning
fn block_on<F: Future>(future: F) -> F::Output {
    // Pin the future on the stack
    let mut future = pin!(future);

    // Create a no-op waker (for demonstration purposes)
    struct NoopWake;
    impl Wake for NoopWake {
        fn wake(self: Arc<Self>) {}
    }
    let waker = Arc::new(NoopWake).into();
    let mut cx = Context::from_waker(&waker);
    
    loop {
        match future.as_mut().poll(&mut cx) {
            Poll::Ready(output) => return output,
            Poll::Pending => { /* In real code: park thread */ }
        }
    }
}
```


## Resources

- [pin-project crate](https://docs.rs/pin-project/latest/pin_project/) — Safe pin projections for custom futures
- [std::pin module](https://doc.rust-lang.org/std/pin/index.html) — Official documentation for Pin, pin!, and related types

---

> 📘 *This lesson is part of the [Rust Ownership & Memory Model](https://stanza.dev/courses/rust-ownership) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*