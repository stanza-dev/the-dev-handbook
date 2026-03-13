---
source_course: "rust-async"
source_lesson: "rust-async-executors"
---

# Understanding Executors

## Introduction

Unlike Go or JavaScript, Rust ships **no built-in async runtime**. An executor is a separate library that drives futures to completion by polling them. This is a deliberate design choice for zero-cost abstraction.

## Key Concepts

**Executor responsibilities:**
1. Polling futures when they're ready
2. Scheduling tasks based on Waker notifications
3. Managing I/O integration (epoll/kqueue/IOCP)
4. Thread pool management

**Popular runtimes:**

| Runtime | Use Case | Threading |
|---------|----------|----------|
| **Tokio** | General purpose, networking | Work-stealing, multi-thread |
| **async-std** | std-like API | Multi-thread |
| **smol** | Minimal, lightweight | Single or multi-thread |
| **Embassy** | Embedded / no_std | Cooperative, single-thread |

## Real World Context

Tokio dominates the ecosystem — most async libraries (Axum, sqlx, reqwest, tonic) are built on it. Choose your runtime early because mixing runtimes in one binary is painful.

## Deep Dive

The executor loop (simplified):

```rust
loop {
    let ready_tasks = wait_for_wakeups();
    for task in ready_tasks {
        match task.poll() {
            Poll::Ready(result) => complete_task(task, result),
            Poll::Pending => {} // Will be woken later
        }
    }
}
```

The key insight: the executor only polls tasks that have been woken. It does NOT busy-loop over all tasks.

## Common Pitfalls

- Using `block_on` inside an async context — deadlocks the executor
- Blocking the executor thread with sync I/O or CPU work
- Assuming all runtimes are interchangeable — they aren't

## Best Practices

- Pick one runtime and stick with it
- Use `spawn_blocking` for CPU-bound or sync blocking work
- Keep executor threads free for I/O polling

## Summary

Rust has no built-in runtime by design. Executors poll futures, manage wakers, and integrate with OS I/O. Tokio is the de facto standard. Never block the executor thread.

## Code Examples

**A simplified block_on executor to show the polling loop**

```rust
// Minimal conceptual executor (educational only)
use std::future::Future;
use std::task::{Context, Poll, Waker};
use std::pin::Pin;

fn block_on<F: Future>(mut future: F) -> F::Output {
    let waker = Waker::noop(); // Rust 1.85+
    let mut cx = Context::from_waker(&waker);
    let mut future = unsafe { Pin::new_unchecked(&mut future) };

    loop {
        match future.as_mut().poll(&mut cx) {
            Poll::Ready(output) => return output,
            Poll::Pending => std::thread::yield_now(),
        }
    }
}
```


## Resources

- [Async Executors](https://rust-lang.github.io/async-book/02_execution/04_executor.html) — Async Book chapter on executors

---

> 📘 *This lesson is part of the [Asynchronous Rust](https://stanza.dev/courses/rust-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*