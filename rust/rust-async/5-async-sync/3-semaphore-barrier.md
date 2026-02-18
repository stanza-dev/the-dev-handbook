---
source_course: "rust-async"
source_lesson: "rust-async-semaphore-barrier"
---

# Advanced Synchronization Primitives

## Semaphore: Limit Concurrency

```rust
use tokio::sync::Semaphore;
use std::sync::Arc;

let semaphore = Arc::new(Semaphore::new(3)); // Max 3 concurrent

for i in 0..10 {
    let permit = semaphore.clone().acquire_owned().await.unwrap();
    tokio::spawn(async move {
        do_work(i).await;
        drop(permit); // Release when done
    });
}
```

## Barrier: Synchronization Point

```rust
use tokio::sync::Barrier;
use std::sync::Arc;

let barrier = Arc::new(Barrier::new(3));

for i in 0..3 {
    let b = barrier.clone();
    tokio::spawn(async move {
        println!("Task {i} before barrier");
        b.wait().await; // All tasks wait here
        println!("Task {i} after barrier");
    });
}
```

## Notify: Event Signaling

```rust
use tokio::sync::Notify;
use std::sync::Arc;

let notify = Arc::new(Notify::new());

// Waiter
let n = notify.clone();
tokio::spawn(async move {
    n.notified().await;
    println!("Received notification!");
});

// Later...
notify.notify_one();
// Or notify all waiters:
notify.notify_waiters();
```

## OnceLock: Async Initialization

```rust
use tokio::sync::OnceCell;

static CONFIG: OnceCell<Config> = OnceCell::const_new();

async fn get_config() -> &'static Config {
    CONFIG.get_or_init(|| async {
        load_config().await
    }).await
}
```

## Code Examples

**Rate limiting with Semaphore**

```rust
use tokio::sync::Semaphore;

// Rate limiter with semaphore
struct RateLimiter {
    semaphore: Semaphore,
}

impl RateLimiter {
    fn new(max_concurrent: usize) -> Self {
        Self {
            semaphore: Semaphore::new(max_concurrent),
        }
    }
    
    async fn acquire(&self) -> tokio::sync::SemaphorePermit<'_> {
        self.semaphore.acquire().await.unwrap()
    }
}

// Usage
let limiter = RateLimiter::new(10);
let _permit = limiter.acquire().await;
do_limited_operation().await;
```


---

> 📘 *This lesson is part of the [Asynchronous Rust](https://stanza.dev/courses/rust-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*