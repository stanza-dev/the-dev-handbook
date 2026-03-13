---
source_course: "rust-concurrency"
source_lesson: "rust-concurrency-mutex-basics"
---

# Mutex<T>: Mutual Exclusion

## Introduction

When multiple threads need to read and write the same data, you need mutual exclusion. Rust's `Mutex<T>` wraps a value and ensures only one thread can access it at a time. Combined with `Arc` for shared ownership, `Arc<Mutex<T>>` is the standard pattern for shared mutable state across threads.

## Key Concepts

- **Mutex<T>**: A mutual exclusion lock that wraps a value of type `T`. Only one thread can hold the lock at a time. All access goes through the lock — there is no way to reach the inner value without locking.
- **MutexGuard<T>**: The RAII guard returned by `lock()`. It implements `Deref` and `DerefMut`, giving you access to the inner value. The lock is released automatically when the guard is dropped.
- **Arc<Mutex<T>>**: `Mutex` itself is not `Clone`. To share a `Mutex` across threads, wrap it in `Arc` (Atomic Reference Counting). Each thread clones the `Arc`, and all clones point to the same `Mutex`.
- **Poisoning**: If a thread panics while holding a `MutexGuard`, the `Mutex` becomes "poisoned". Subsequent calls to `lock()` return `Err(PoisonError)`, signaling that the protected data may be in an inconsistent state.
- **try_lock()**: A non-blocking alternative to `lock()`. Returns `Ok(guard)` if the lock is available, or `Err(TryLockError)` if another thread holds it or the mutex is poisoned.

## Real World Context

Shared counters, connection pools, caches, and configuration stores are all common use cases for `Mutex`. A web server might use `Arc<Mutex<HashMap<String, Session>>>` to store session data accessible from any request handler thread. Game engines use mutexes to protect shared world state that multiple systems (physics, rendering, AI) need to update.

## Deep Dive

A `Mutex` guards its inner value by requiring you to call `lock()` before accessing it. The returned `MutexGuard` gives you `&mut T` access and automatically releases the lock when dropped.

```rust
use std::sync::Mutex;

let m = Mutex::new(5);

{
    let mut guard = m.lock().unwrap();
    *guard += 1;
    println!("value: {}", *guard); // 6
} // guard dropped here — lock released

// Lock again to read
println!("final: {}", *m.lock().unwrap()); // 6
```

To share a `Mutex` between threads, wrap it in `Arc`. Each thread gets its own `Arc` clone that points to the same underlying `Mutex`.

```rust
use std::sync::{Arc, Mutex};
use std::thread;

let counter = Arc::new(Mutex::new(0));
let mut handles = vec![];

for _ in 0..10 {
    let counter = Arc::clone(&counter);
    let handle = thread::spawn(move || {
        let mut num = counter.lock().unwrap();
        *num += 1;
    });
    handles.push(handle);
}

for handle in handles {
    handle.join().unwrap();
}

println!("final count: {}", *counter.lock().unwrap()); // 10
```

The `try_lock()` method attempts to acquire the lock without blocking. This is useful when you want to do other work if the lock is not immediately available.

```rust
use std::sync::Mutex;

let m = Mutex::new(42);

match m.try_lock() {
    Ok(guard) => println!("got the lock: {}", *guard),
    Err(_) => println!("lock is held by another thread, doing other work"),
}
```

When a thread panics while holding a `MutexGuard`, the mutex becomes poisoned. This is a safety mechanism — the data inside might be in a half-updated, inconsistent state. You can choose to handle poisoning or propagate it.

```rust
use std::sync::{Arc, Mutex};
use std::thread;

let data = Arc::new(Mutex::new(vec![1, 2, 3]));
let data_clone = Arc::clone(&data);

let _ = thread::spawn(move || {
    let mut guard = data_clone.lock().unwrap();
    guard.push(4);
    panic!("oops!"); // guard is held when panic occurs — mutex is poisoned
}).join();

// Subsequent lock attempts return Err(PoisonError)
match data.lock() {
    Ok(guard) => println!("data: {:?}", *guard),
    Err(poison_err) => {
        // Recover the data if you believe it's still valid
        let guard = poison_err.into_inner();
        println!("recovered poisoned data: {:?}", *guard);
    }
}
```

You can also clear the poisoned state so future callers do not see the error.

```rust
use std::sync::Mutex;

let m = Mutex::new(0);
// ... after poisoning ...
m.clear_poison();
// Now lock() returns Ok again
```

A common pattern is to hold the lock for the shortest time possible. Avoid doing I/O or expensive computation while holding a `MutexGuard` — other threads will block waiting for you.

```rust
use std::sync::{Arc, Mutex};

let shared_config = Arc::new(Mutex::new(load_config()));

// Good: lock briefly, clone the data, release immediately
let config_snapshot = {
    let guard = shared_config.lock().unwrap();
    guard.clone()
}; // lock released here

// Work with the snapshot without holding the lock
process_config(&config_snapshot);
```

## Common Pitfalls

1. **Deadlocks from nested locking** — If a thread tries to lock a `Mutex` it already holds, it will deadlock because `Mutex` in Rust is not reentrant. Always ensure you drop the guard before locking the same mutex again.
2. **Holding locks across `.await` or long operations** — Holding a `MutexGuard` across an `.await` point in async code or during I/O starves other threads. Extract the data you need, drop the guard, then do the slow work.

## Best Practices

1. **Minimize lock scope** — Use a block `{ let guard = m.lock().unwrap(); ... }` to ensure the guard is dropped as soon as possible. This reduces contention and deadlock risk.
2. **Use `Mutex::new()` with interior mutability in mind** — `Mutex` provides interior mutability: even if the `Mutex` itself is behind a shared reference (`&Mutex<T>`), you get `&mut T` access through the guard. Design your types to take advantage of this.

## Summary

- `Mutex<T>` ensures only one thread accesses the inner value at a time via an RAII guard.
- `Arc<Mutex<T>>` is the standard pattern for sharing mutable state across threads.
- `try_lock()` provides non-blocking lock acquisition.
- Poisoning protects against inconsistent state after a thread panic; use `PoisonError::into_inner()` to recover if the data is still valid.
- Keep lock scopes as short as possible to minimize contention.

## Code Examples

**Shared counter using Arc<Mutex<T>> across 10 threads**

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Final count: {}", *counter.lock().unwrap());
}
```


## Resources

- [Shared-State Concurrency](https://doc.rust-lang.org/book/ch16-03-shared-state.html) — Official Rust Book chapter on Mutex and shared state
- [std::sync::Mutex](https://doc.rust-lang.org/std/sync/struct.Mutex.html) — Standard library reference for Mutex<T>

---

> 📘 *This lesson is part of the [Concurrent Rust](https://stanza.dev/courses/rust-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*