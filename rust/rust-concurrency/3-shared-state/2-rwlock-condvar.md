---
source_course: "rust-concurrency"
source_lesson: "rust-concurrency-rwlock-condvar"
---

# RwLock & Condvar

## Introduction

Not all shared state access patterns are equal. When data is read far more often than it is written, `Mutex` forces unnecessary serialization — every reader waits for every other reader. `RwLock` solves this by allowing multiple simultaneous readers or one exclusive writer. `Condvar` (condition variable) complements both lock types by letting threads efficiently wait for a specific condition to become true.

## Key Concepts

- **RwLock<T>**: A reader-writer lock. Multiple threads can hold `read()` locks simultaneously, but `write()` requires exclusive access. Ideal for read-heavy workloads.
- **RwLockReadGuard / RwLockWriteGuard**: RAII guards returned by `read()` and `write()`. The read guard implements `Deref` (shared access), and the write guard implements both `Deref` and `DerefMut` (exclusive access).
- **RwLockWriteGuard::downgrade()**: Stabilized in Rust 1.92, this method atomically downgrades a write lock to a read lock without releasing it. No other writer can acquire the lock between the downgrade, preventing TOCTOU (time-of-check-to-time-of-use) races.
- **Condvar**: A condition variable that allows threads to block until a particular condition is signaled. Used in combination with a `Mutex` — the thread locks the mutex, checks a condition, and calls `wait()` if the condition is not met. `wait()` atomically releases the mutex and puts the thread to sleep.
- **Spurious Wakeups**: `Condvar::wait()` may return even without a corresponding `notify`. Always check the condition in a loop.

## Real World Context

Configuration stores, routing tables, and DNS caches are classic `RwLock` use cases — they are read on every request but updated rarely. The new `downgrade()` method is valuable when you need to initialize or update data under a write lock and then immediately read it without giving another writer a chance to intervene. Condition variables power producer-consumer queues, thread pools, and shutdown coordination: workers wait on a condvar for new tasks, and the producer notifies when work is available.

## Deep Dive

### RwLock Basics

`RwLock` allows many readers or one writer, but never both at the same time.

```rust
use std::sync::RwLock;

let lock = RwLock::new(5);

// Multiple readers can acquire the lock simultaneously
{
    let r1 = lock.read().unwrap();
    let r2 = lock.read().unwrap();
    println!("readers see: {} and {}", *r1, *r2);
} // both read guards dropped

// Only one writer at a time, and no readers while writing
{
    let mut w = lock.write().unwrap();
    *w += 1;
    println!("writer set value to: {}", *w);
} // write guard dropped
```

Sharing an `RwLock` across threads follows the same `Arc` pattern as `Mutex`.

```rust
use std::sync::{Arc, RwLock};
use std::thread;

let config = Arc::new(RwLock::new(vec!["default".to_string()]));

// Spawn 5 reader threads
let readers: Vec<_> = (0..5)
    .map(|i| {
        let config = Arc::clone(&config);
        thread::spawn(move || {
            let data = config.read().unwrap();
            println!("reader {i}: {:?}", *data);
        })
    })
    .collect();

// Spawn 1 writer thread
let config_w = Arc::clone(&config);
let writer = thread::spawn(move || {
    let mut data = config_w.write().unwrap();
    data.push("updated".to_string());
    println!("writer updated config");
});

for r in readers {
    r.join().unwrap();
}
writer.join().unwrap();
```

### RwLockWriteGuard::downgrade() (Rust 1.92+)

The `downgrade()` method converts a write guard into a read guard atomically. This is critical when you need to perform a write and then immediately read the result without allowing another writer to change the data in between.

```rust
use std::sync::{Arc, RwLock, RwLockWriteGuard};
use std::thread;

let lock = Arc::new(RwLock::new(String::new()));

let lock_clone = Arc::clone(&lock);
let handle = thread::spawn(move || {
    // Acquire exclusive write access
    let mut write_guard = lock_clone.write().unwrap();
    write_guard.push_str("initialized");

    // Downgrade to a read lock — no other writer can intervene
    let read_guard = RwLockWriteGuard::downgrade(write_guard);

    // Other readers can now acquire the lock concurrently
    println!("after init: {}", *read_guard);
    // read_guard is dropped here, releasing the read lock
});

handle.join().unwrap();
println!("final: {}", *lock.read().unwrap());
```

Without `downgrade()`, you would have to drop the write guard and re-acquire a read guard, creating a window where another thread could acquire a write lock and change the data.

### Condvar: Condition Variables

A `Condvar` lets threads sleep until a condition is met. It is always paired with a `Mutex` that protects the condition state.

```rust
use std::sync::{Arc, Condvar, Mutex};
use std::thread;

let pair = Arc::new((Mutex::new(false), Condvar::new()));
let pair_clone = Arc::clone(&pair);

// Waiting thread
let waiter = thread::spawn(move || {
    let (lock, cvar) = &*pair_clone;
    let mut ready = lock.lock().unwrap();

    // Always check in a loop — spurious wakeups are possible
    while !*ready {
        ready = cvar.wait(ready).unwrap();
    }

    println!("condition met, proceeding!");
});

// Signaling thread
let (lock, cvar) = &*pair;
{
    let mut ready = lock.lock().unwrap();
    *ready = true;
}
cvar.notify_one(); // wake one waiting thread

waiter.join().unwrap();
```

### Producer-Consumer with Condvar

A classic producer-consumer queue uses a `Mutex<VecDeque>` with a `Condvar` to signal when items are available.

```rust
use std::collections::VecDeque;
use std::sync::{Arc, Condvar, Mutex};
use std::thread;

struct WorkQueue<T> {
    queue: Mutex<VecDeque<T>>,
    condvar: Condvar,
}

impl<T> WorkQueue<T> {
    fn new() -> Self {
        Self {
            queue: Mutex::new(VecDeque::new()),
            condvar: Condvar::new(),
        }
    }

    fn push(&self, item: T) {
        let mut queue = self.queue.lock().unwrap();
        queue.push_back(item);
        self.condvar.notify_one();
    }

    fn pop(&self) -> T {
        let mut queue = self.queue.lock().unwrap();
        while queue.is_empty() {
            queue = self.condvar.wait(queue).unwrap();
        }
        queue.pop_front().unwrap()
    }
}

let wq = Arc::new(WorkQueue::new());
let wq_consumer = Arc::clone(&wq);

let consumer = thread::spawn(move || {
    for _ in 0..5 {
        let item: i32 = wq_consumer.pop();
        println!("consumed: {item}");
    }
});

for i in 0..5 {
    wq.push(i);
    thread::sleep(std::time::Duration::from_millis(100));
}

consumer.join().unwrap();
```

`notify_one()` wakes a single waiting thread. Use `notify_all()` when multiple threads might be interested in the same condition change (for example, a shutdown signal).

```rust
use std::sync::{Arc, Condvar, Mutex};

let shutdown = Arc::new((Mutex::new(false), Condvar::new()));

// When shutting down:
{
    let (lock, cvar) = &*shutdown;
    *lock.lock().unwrap() = true;
    cvar.notify_all(); // wake ALL waiting threads
}
```

## Common Pitfalls

1. **Using RwLock for write-heavy workloads** — `RwLock` has higher overhead than `Mutex` because it tracks reader counts. If writes are as frequent as reads, `Mutex` is simpler and often faster.
2. **Not looping on `Condvar::wait()`** — Condition variables are subject to spurious wakeups. Always recheck the condition in a `while` loop after `wait()` returns.

## Best Practices

1. **Use `RwLock` when reads vastly outnumber writes** — The break-even point varies by platform, but a general guideline is to prefer `RwLock` when your workload is at least 80% reads.
2. **Use `downgrade()` to avoid TOCTOU races** — When you need to write then immediately read, `RwLockWriteGuard::downgrade()` (Rust 1.92+) prevents another writer from intervening between the write and the read.

## Summary

- `RwLock<T>` allows multiple concurrent readers or one exclusive writer.
- `RwLockWriteGuard::downgrade()` (Rust 1.92) atomically converts a write lock to a read lock without releasing it.
- `Condvar` lets threads sleep until a condition is signaled; always check conditions in a loop to handle spurious wakeups.
- The producer-consumer queue pattern combines `Mutex<VecDeque<T>>` with `Condvar` for efficient work distribution.

## Code Examples

**RwLock with write-to-read downgrade (Rust 1.92+) and concurrent readers**

```rust
use std::sync::{Arc, RwLock, RwLockWriteGuard};
use std::thread;

fn main() {
    let data = Arc::new(RwLock::new(Vec::<String>::new()));

    // Writer initializes data, then downgrades to reader
    let data_w = Arc::clone(&data);
    let writer = thread::spawn(move || {
        let mut wg = data_w.write().unwrap();
        wg.push("initialized".to_string());

        // Downgrade: write -> read, no gap for other writers
        let rg = RwLockWriteGuard::downgrade(wg);
        println!("Writer (now reader): {:?}", *rg);
    });

    // Multiple concurrent readers
    let readers: Vec<_> = (0..3)
        .map(|i| {
            let data = Arc::clone(&data);
            thread::spawn(move || {
                let rg = data.read().unwrap();
                println!("Reader {i}: {:?}", *rg);
            })
        })
        .collect();

    writer.join().unwrap();
    for r in readers {
        r.join().unwrap();
    }
}
```


## Resources

- [std::sync::RwLock](https://doc.rust-lang.org/std/sync/struct.RwLock.html) — Standard library reference for RwLock<T>
- [std::sync::Condvar](https://doc.rust-lang.org/std/sync/struct.Condvar.html) — Standard library reference for condition variables
- [RwLockWriteGuard::downgrade](https://doc.rust-lang.org/std/sync/struct.RwLockWriteGuard.html#method.downgrade) — Write-to-read downgrade stabilized in Rust 1.92

---

> 📘 *This lesson is part of the [Concurrent Rust](https://stanza.dev/courses/rust-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*