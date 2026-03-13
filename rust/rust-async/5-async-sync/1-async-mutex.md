---
source_course: "rust-async"
source_lesson: "rust-async-mutex"
---

# Async vs Std Mutex

## Introduction

Holding a `std::sync::Mutex` guard across `.await` points blocks the executor thread, starving other tasks. `tokio::sync::Mutex` yields instead of blocking, and `tokio::sync::RwLock` adds read/write separation for async code.

## Key Concepts

**The problem** — std Mutex blocks the thread:
```rust
let lock = std_mutex.lock().unwrap();
do_something().await; // BAD! Thread is blocked
```

**The solution** — async Mutex yields:
```rust
use tokio::sync::Mutex;
let mut lock = async_mutex.lock().await; // Yields, not blocks
*lock += 1;
some_async_op().await; // OK to hold across await
```

## Real World Context

Shared caches, connection pools, rate limiters, and any shared mutable state accessed from async tasks need either an async mutex or careful scoping of std mutex guards.

## Deep Dive

**When to use which:**

| Scenario | Use |
|----------|-----|
| Short sync-only critical section | `std::sync::Mutex` |
| Lock held across `.await` | `tokio::sync::Mutex` |
| Read-heavy workloads | `tokio::sync::RwLock` |
| No await needed | `std::sync::Mutex` (faster) |

**std Mutex is fine if you drop before await:**
```rust
let value = {
    let lock = std_mutex.lock().unwrap();
    lock.clone()
}; // Lock dropped here
do_something(value).await; // Safe!
```

**RwLock** for read-heavy patterns:
```rust
use tokio::sync::RwLock;
let lock = RwLock::new(vec![1, 2, 3]);
let r = lock.read().await;  // Multiple readers OK
let mut w = lock.write().await; // Exclusive writer
```

## Common Pitfalls

- Holding std Mutex across `.await` — blocks the executor thread
- Using async Mutex everywhere — std Mutex is faster for short critical sections
- Deadlocking by acquiring multiple locks in inconsistent order

## Best Practices

- Default to `std::sync::Mutex` and drop before `.await`
- Only use `tokio::sync::Mutex` when you must hold across awaits
- Wrap in `Arc` for sharing between spawned tasks

## Summary

std Mutex blocks threads; tokio Mutex yields. Use std Mutex for short sections dropped before await. Use async Mutex when holding across await points. RwLock optimizes read-heavy patterns.

## Code Examples

**Shared mutable state with Arc<Mutex<T>> across tasks**

```rust
use std::sync::Arc;
use tokio::sync::Mutex;

#[tokio::main]
async fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        handles.push(tokio::spawn(async move {
            let mut lock = counter.lock().await;
            *lock += 1;
        }));
    }

    for h in handles { h.await.unwrap(); }
    println!("Count: {}", *counter.lock().await);
}
```


## Resources

- [Shared State](https://tokio.rs/tokio/tutorial/shared-state) — Tokio tutorial on shared state with async mutex

---

> 📘 *This lesson is part of the [Asynchronous Rust](https://stanza.dev/courses/rust-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*