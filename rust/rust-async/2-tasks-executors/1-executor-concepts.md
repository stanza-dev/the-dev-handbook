---
source_course: "rust-async"
source_lesson: "rust-async-executor-concepts"
---

# What is an Executor?

An executor is the runtime that drives Futures to completion.

## Core Responsibilities

1. **Polling Futures** - Calls `poll()` on tasks
2. **Task Scheduling** - Decides which task runs next
3. **Waker Management** - Tracks which tasks are ready
4. **I/O Integration** - Connects to OS async I/O (epoll, kqueue, IOCP)

## Why Rust Needs External Runtimes

Unlike Go or JavaScript, Rust doesn't include a runtime. This is intentional:

- **Zero-cost abstraction** - Pay only for what you use
- **Flexibility** - Choose the right runtime for your use case
- **No_std support** - Embedded systems can use async

## Popular Runtimes

| Runtime | Use Case | Scheduling |
|---------|----------|------------|
| **Tokio** | General purpose, networking | Work-stealing, multi-threaded |
| **async-std** | std-like API | Multi-threaded |
| **smol** | Minimal, lightweight | Single or multi-threaded |
| **Embassy** | Embedded systems | Cooperative, no_std |

## Executor Loop (Simplified)

```rust
loop {
    // Wait for any task to become ready
    let ready_tasks = wait_for_wakeups();
    
    for task in ready_tasks {
        match task.poll() {
            Poll::Ready(result) => complete_task(task, result),
            Poll::Pending => continue, // Will be woken later
        }
    }
}
```

## Code Examples

**Simplified executor concept**

```rust
// Minimal executor concept (educational)
use std::future::Future;
use std::task::{Context, Poll, RawWaker, RawWakerVTable, Waker};
use std::pin::Pin;

fn block_on<F: Future>(mut future: F) -> F::Output {
    let waker = dummy_waker();
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


---

> 📘 *This lesson is part of the [Asynchronous Rust](https://stanza.dev/courses/rust-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*