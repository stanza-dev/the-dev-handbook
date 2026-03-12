---
source_course: "rust-ownership"
source_lesson: "rust-ownership-mutex-rwlock"
---

# Mutex<T> & RwLock<T>: Thread-Safe Interior Mutability

## Introduction
Cell and RefCell are single-threaded. When you need interior mutability across threads, Rust provides Mutex (mutual exclusion) and RwLock (read-write lock). These types block the current thread until access is available, ensuring data race freedom.

## Key Concepts
- **Mutex<T>**: A lock that provides exclusive access to the inner value. Only one thread can hold the lock at a time.
- **RwLock<T>**: A lock that allows multiple simultaneous readers or one exclusive writer.
- **MutexGuard / RwLockReadGuard / RwLockWriteGuard**: RAII guards that release the lock when dropped.

## Real World Context
Every multi-threaded server, worker pool, or parallel pipeline uses some form of locking. A shared request counter uses `Arc<Mutex<usize>>`. A configuration store read by many threads but rarely updated uses `Arc<RwLock<Config>>`. These patterns appear in virtually every production Rust application.

## Deep Dive
Mutex provides exclusive access via a lock:

```rust
use std::sync::Mutex;

let m = Mutex::new(5);

{
    let mut guard = m.lock().unwrap();
    *guard = 10;
}  // Lock released when guard drops

println!("{:?}", m.lock().unwrap());  // 10
```

The `Arc<Mutex<T>>` pattern is the thread-safe equivalent of `Rc<RefCell<T>>`:

```rust
use std::sync::{Arc, Mutex};
use std::thread;

let counter = Arc::new(Mutex::new(0));
let mut handles = vec![];

for _ in 0..10 {
    let counter = Arc::clone(&counter);
    handles.push(thread::spawn(move || {
        let mut num = counter.lock().unwrap();
        *num += 1;
    }));
}

for handle in handles { handle.join().unwrap(); }
println!("Count: {}", *counter.lock().unwrap());  // 10
```

RwLock allows multiple concurrent readers:

```rust
use std::sync::RwLock;

let lock = RwLock::new(5);

// Multiple readers OK
let r1 = lock.read().unwrap();
let r2 = lock.read().unwrap();
drop(r1); drop(r2);

// Writer needs exclusive access
let mut w = lock.write().unwrap();
*w += 1;
```

### LazyLock: Thread-Safe Lazy Initialization

Since Rust 1.80, `LazyLock` provides thread-safe lazy initialization, replacing the common `OnceLock` + initialization pattern and the third-party `lazy_static` and `once_cell` crates:

```rust
use std::sync::LazyLock;

static CONFIG: LazyLock<String> = LazyLock::new(|| {
    std::fs::read_to_string("/etc/app.conf")
        .unwrap_or_default()
});

// First access triggers initialization (thread-safe)
println!("{}", *CONFIG);
```

LazyLock is `Sync`, so it can be used in `static` items and shared across threads safely. For single-threaded lazy initialization, use `LazyCell` instead.

## Common Pitfalls
1. **Deadlocking by locking twice on the same thread** — Mutex is not reentrant. If you call `lock()` while already holding the lock on the same thread, it deadlocks forever.
2. **Ignoring poisoned locks** — If a thread panics while holding a lock, the lock becomes "poisoned". Using `unwrap()` on a poisoned lock will also panic. Handle this with `lock().unwrap_or_else(|e| e.into_inner())`.

## Best Practices
1. **Hold locks for as short a time as possible** — Lock, do your work, drop the guard. Never hold a lock across an await point or long computation.
2. **Use RwLock for read-heavy workloads** — When reads vastly outnumber writes, RwLock's concurrent reader access provides better throughput than Mutex.
3. **Use LazyLock instead of `lazy_static!` or `once_cell`** — LazyLock is now in the standard library and should be preferred over third-party alternatives for new code.

## Summary
- Mutex provides exclusive access; RwLock allows multiple readers or one writer.
- Both are thread-safe and block until the lock is available.
- The Arc<Mutex<T>> pattern is the standard way to share mutable state across threads.
- LazyLock (Rust 1.80+) provides thread-safe lazy initialization, replacing `lazy_static!` and `once_cell`.
- Drop lock guards as quickly as possible to minimize contention.

## Code Examples

**RwLock for shared config — multiple readers access concurrently, writers get exclusive access**

```rust
use std::sync::{Arc, RwLock};
use std::thread;

// Read-heavy configuration shared across threads
struct AppConfig {
    max_retries: usize,
    timeout_ms: u64,
}

let config = Arc::new(RwLock::new(AppConfig { max_retries: 3, timeout_ms: 5000 }));

// Many readers can access simultaneously
for i in 0..10 {
    let config = Arc::clone(&config);
    thread::spawn(move || {
        let cfg = config.read().unwrap();
        println!("Reader {i}: timeout={}", cfg.timeout_ms);
    });
}

// Occasional writer gets exclusive access
{
    let mut cfg = config.write().unwrap();
    cfg.timeout_ms = 10000;
}
```


## Resources

- [Mutex Documentation](https://doc.rust-lang.org/std/sync/struct.Mutex.html) — Official API reference for Mutex
- [RwLock Documentation](https://doc.rust-lang.org/std/sync/struct.RwLock.html) — Official API reference for RwLock

---

> 📘 *This lesson is part of the [Rust Ownership & Memory Model](https://stanza.dev/courses/rust-ownership) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*