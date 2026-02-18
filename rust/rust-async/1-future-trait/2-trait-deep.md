---
source_course: "rust-async"
source_lesson: "rust-async-future-trait-deep"
---

# The Future Trait

Understanding the trait is key to mastering async Rust.

```rust
pub trait Future {
    type Output;
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}

pub enum Poll<T> {
    Ready(T),
    Pending,
}
```

## How Polling Works

1. An **Executor** calls `poll()` on your Future
2. If ready, returns `Poll::Ready(value)`
3. If not ready:
   - Registers a `Waker` with the async resource
   - Returns `Poll::Pending`
4. When the resource completes, it calls `waker.wake()`
5. The Executor polls again

## The Waker Mechanism

The `Waker` is how async I/O notifies the executor that progress can be made.

```rust
impl Future for MyTimer {
    type Output = ();
    
    fn poll(self: Pin<&mut Self>, cx: &mut Context) -> Poll<()> {
        if self.is_ready() {
            Poll::Ready(())
        } else {
            // Register the waker so we get polled again
            self.register_waker(cx.waker().clone());
            Poll::Pending
        }
    }
}
```

## Why Pin?

Async functions compile to state machines that may contain self-references. `Pin` prevents moving the Future in memory, which would invalidate those references.

```rust
async fn example() {
    let data = vec![1, 2, 3];
    let reference = &data[0]; // Self-reference!
    some_async_op().await;    // State machine stores both
    println!("{reference}");
}
```

See [The Future Trait](https://rust-lang.github.io/async-book/02_execution/02_future.html).

## Code Examples

**Implementing a simple Future**

```rust
use std::future::Future;
use std::pin::Pin;
use std::task::{Context, Poll};

// A Future that's immediately ready
struct Ready<T>(Option<T>);

impl<T> Future for Ready<T> {
    type Output = T;
    
    fn poll(mut self: Pin<&mut Self>, _cx: &mut Context) -> Poll<T> {
        Poll::Ready(self.0.take().unwrap())
    }
}

// Usage
async fn example() {
    let value = Ready(Some(42)).await;
    assert_eq!(value, 42);
}
```


---

> 📘 *This lesson is part of the [Asynchronous Rust](https://stanza.dev/courses/rust-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*