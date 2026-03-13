---
source_course: "rust-async"
source_lesson: "rust-async-future-internals"
---

# The Future Trait Internals

## Introduction

Every async operation in Rust ultimately implements the `Future` trait. Understanding `poll`, `Pin`, `Context`, and the `Waker` mechanism is essential for writing custom futures and debugging async code.

## Key Concepts

The `Future` trait definition:

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

**Poll::Ready(value)** — the future has completed with a value.
**Poll::Pending** — the future is not yet complete; the executor should not poll again until woken.

## Real World Context

You rarely implement `Future` by hand — `async fn` does it for you. But understanding the trait is critical when writing custom combinators, working with `Pin`, or debugging executor behavior.

## Deep Dive

**The polling cycle:**
1. Executor calls `poll()` on the future
2. If ready, returns `Poll::Ready(value)`
3. If not ready, registers a `Waker` with the I/O resource and returns `Poll::Pending`
4. When the resource completes, it calls `waker.wake()`
5. Executor schedules the task to be polled again

**The Waker mechanism** is how async I/O notifies the executor. Since Rust 1.83, you can construct wakers with `Waker::new()` using data + vtable. Since 1.85, `Waker::noop()` provides a no-op waker for testing.

```rust
impl Future for MyTimer {
    type Output = ();
    fn poll(self: Pin<&mut Self>, cx: &mut Context) -> Poll<()> {
        if self.is_ready() {
            Poll::Ready(())
        } else {
            self.register_waker(cx.waker().clone());
            Poll::Pending
        }
    }
}
```

**Why Pin?** Async state machines may contain self-references (a local variable referencing another local). `Pin` prevents moving the future in memory, which would invalidate those references.

## Common Pitfalls

- Returning `Pending` without registering a waker — the future never gets polled again
- Calling `waker.wake()` too eagerly — causes busy-polling
- Ignoring `Pin` requirements when storing futures in collections

## Best Practices

- Always register or clone the waker before returning `Pending`
- Use `Waker::noop()` (1.85+) for unit testing custom futures
- Prefer `async fn` over manual `Future` implementations unless you need fine-grained control

## Summary

`Future::poll` returns `Ready` or `Pending`. The `Waker` notifies the executor when a task can make progress. `Pin` protects self-referential state machines from being moved in memory.

## Code Examples

**Implementing a minimal Future by hand**

```rust
use std::future::Future;
use std::pin::Pin;
use std::task::{Context, Poll};

// A future that is immediately ready
struct Ready<T>(Option<T>);

impl<T> Future for Ready<T> {
    type Output = T;
    fn poll(mut self: Pin<&mut Self>, _cx: &mut Context) -> Poll<T> {
        Poll::Ready(self.0.take().unwrap())
    }
}

async fn example() {
    let value = Ready(Some(42)).await;
    assert_eq!(value, 42);
}
```


## Resources

- [The Future Trait](https://rust-lang.github.io/async-book/02_execution/02_future.html) — Async Book chapter on the Future trait

---

> 📘 *This lesson is part of the [Asynchronous Rust](https://stanza.dev/courses/rust-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*