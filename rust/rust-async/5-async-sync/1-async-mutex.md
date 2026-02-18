---
source_course: "rust-async"
source_lesson: "rust-async-async-mutex"
---

# The Problem with std::sync::Mutex

Holding a standard Mutex guard across `.await` blocks the entire thread!

```rust
let lock = std_mutex.lock().unwrap();
do_something().await; // BAD! Thread blocked while holding lock
```

**Why?** The std Mutex guard is `!Send`, and blocking on it wastes the thread.

## tokio::sync::Mutex

Async-aware mutex that yields instead of blocking:

```rust
use tokio::sync::Mutex;

let mutex = Mutex::new(0);

async fn increment(m: &Mutex<i32>) {
    let mut lock = m.lock().await; // Yields, doesn't block thread
    *lock += 1;
    // Lock can be held across await points
    some_async_work().await;
    *lock += 1;
}
```

## When to Use Which?

| Scenario | Use |
|----------|-----|
| Short, sync operations | `std::sync::Mutex` |
| Lock held across `.await` | `tokio::sync::Mutex` |
| High contention, async | `tokio::sync::Mutex` |
| Performance-critical, no await | `std::sync::Mutex` |

**Important:** `std::sync::Mutex` is often fine if you drop the lock before `.await`:

```rust
let value = {
    let lock = std_mutex.lock().unwrap();
    lock.clone()
}; // Lock dropped here
do_something(value).await; // OK!
```

## RwLock for Read-Heavy Workloads

```rust
use tokio::sync::RwLock;

let lock = RwLock::new(vec![1, 2, 3]);

// Multiple readers OK
let r1 = lock.read().await;
let r2 = lock.read().await;

// Writer needs exclusive access
let mut w = lock.write().await;
w.push(4);
```

See [Shared State](https://tokio.rs/tokio/tutorial/shared-state).

## Code Examples

**Shared mutable state with async Mutex**

```rust
use std::sync::Arc;
use tokio::sync::Mutex;

#[tokio::main]
async fn main() {
    let data = Arc::new(Mutex::new(vec![]));
    
    let mut handles = vec![];
    
    for i in 0..10 {
        let data = Arc::clone(&data);
        handles.push(tokio::spawn(async move {
            let mut lock = data.lock().await;
            lock.push(i);
        }));
    }
    
    for handle in handles {
        handle.await.unwrap();
    }
    
    println!("{:?}", data.lock().await);
}
```


---

> 📘 *This lesson is part of the [Asynchronous Rust](https://stanza.dev/courses/rust-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*