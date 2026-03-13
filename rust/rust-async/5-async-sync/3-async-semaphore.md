---
source_course: "rust-async"
source_lesson: "rust-async-semaphore"
---

# Semaphore, Barrier & Notify

## Introduction

Beyond mutexes and channels, Tokio provides `Semaphore` for concurrency limiting, `Barrier` for synchronization points, `Notify` for event signaling, and `OnceCell` for async lazy initialization. These are the building blocks for rate limiters, coordination protocols, and one-time setup.

## Key Concepts

**Semaphore** — limit concurrent access to N:
```rust
use tokio::sync::Semaphore;
let sem = Semaphore::new(3); // Max 3 concurrent
let permit = sem.acquire().await.unwrap();
// ... do work ...
drop(permit); // Release
```

**Barrier** — synchronize N tasks at a point:
```rust
use tokio::sync::Barrier;
let barrier = Barrier::new(3);
// Each task calls: barrier.wait().await;
// All proceed only when 3 tasks have reached the barrier
```

**Notify** — lightweight async event signaling.
**OnceCell** — async lazy one-time initialization.

## Real World Context

Rate limiting HTTP clients (Semaphore), coordinating distributed test setup (Barrier), waking background flushers (Notify), lazy database pool initialization (OnceCell).

## Deep Dive

**Rate limiting pattern:**
```rust
let sem = Arc::new(Semaphore::new(10));
for url in urls {
    let permit = sem.clone().acquire_owned().await.unwrap();
    tokio::spawn(async move {
        let _permit = permit; // Held for duration
        fetch(url).await;
    }); // permit dropped when task ends
}
```

**Notify — signal without data:**
```rust
let notify = Arc::new(tokio::sync::Notify::new());

// Waiter
let n = notify.clone();
tokio::spawn(async move {
    n.notified().await;
    println!("Woken!");
});

notify.notify_one(); // Wake one waiter
notify.notify_waiters(); // Wake all waiters
```

**OnceCell — async lazy init:**
```rust
use tokio::sync::OnceCell;
static DB: OnceCell<Pool> = OnceCell::const_new();

async fn get_db() -> &'static Pool {
    DB.get_or_init(|| async { create_pool().await }).await
}
```

## Common Pitfalls

- Forgetting to drop semaphore permits — tasks never get access
- Barrier with wrong count — tasks hang forever
- Using Notify when you need to buffer signals — use a channel instead

## Best Practices

- Use `acquire_owned()` with `spawn` so the permit moves into the task
- Prefer Semaphore over manual counting for concurrency limits
- Use OnceCell for expensive one-time async initialization

## Summary

Semaphore limits concurrency to N permits. Barrier synchronizes N tasks at a rendezvous point. Notify provides lightweight async signaling. OnceCell enables async lazy initialization. These primitives complement mutexes and channels.

## Code Examples

**Semaphore-based rate limiter for concurrent requests**

```rust
use std::sync::Arc;
use tokio::sync::Semaphore;

// Rate limiter: max 5 concurrent requests
async fn fetch_all(urls: Vec<String>) {
    let sem = Arc::new(Semaphore::new(5));
    let mut handles = vec![];

    for url in urls {
        let permit = sem.clone().acquire_owned().await.unwrap();
        handles.push(tokio::spawn(async move {
            let _permit = permit; // Held until task ends
            reqwest::get(&url).await
        }));
    }

    for h in handles {
        h.await.ok();
    }
}
```


## Resources

- [tokio::sync](https://docs.rs/tokio/latest/tokio/sync/index.html) — Full reference for Tokio synchronization primitives

---

> 📘 *This lesson is part of the [Asynchronous Rust](https://stanza.dev/courses/rust-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*