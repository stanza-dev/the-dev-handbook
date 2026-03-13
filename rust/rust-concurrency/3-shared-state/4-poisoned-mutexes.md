---
source_course: "rust-concurrency"
source_lesson: "rust-concurrency-poisoned-mutexes"
---

# Poisoned Mutexes & Recovery

## Introduction

When a thread panics while holding a `MutexGuard`, the mutex becomes "poisoned." This is Rust's safety mechanism to prevent other threads from accessing data that may be in an inconsistent, half-updated state. Understanding poisoning, when it matters, and how to recover from it is essential for writing robust concurrent Rust code.

## Key Concepts

- **Poisoning**: A mutex becomes poisoned when a thread panics while holding the lock. All subsequent `lock()` calls return `Err(PoisonError)` instead of `Ok(guard)`.
- **PoisonError<T>**: Contains the `MutexGuard` that would have been returned. You can call `into_inner()` to recover the guard and access the data if you believe it is still valid.
- **clear_poison()**: Stabilized in Rust 1.77, this method clears the poisoned state so future `lock()` calls return `Ok` again.
- **parking_lot alternative**: The `parking_lot` crate's `Mutex` does not use poisoning at all, which simplifies code when you do not need this protection.

## Real World Context

In many applications, mutex poisoning is more of a nuisance than a safety feature. If a thread panics while incrementing a counter, the counter value is still valid — it is just one increment behind. But if a thread panics halfway through a multi-step transaction (e.g., transferring money between accounts: debited but not credited), the data truly is inconsistent. Understanding your data's invariants tells you whether to recover from poisoning or propagate the error.

## Deep Dive

Here is how poisoning occurs and how to detect it.

```rust
use std::sync::{Arc, Mutex};
use std::thread;

let data = Arc::new(Mutex::new(vec![1, 2, 3]));
let data_clone = Arc::clone(&data);

// This thread will panic while holding the lock
let _ = thread::spawn(move || {
    let mut guard = data_clone.lock().unwrap();
    guard.push(4);
    panic!("oops!"); // guard is still held — mutex becomes poisoned
}).join(); // join() returns Err, but we ignore it

// Now the mutex is poisoned
assert!(data.lock().is_err());
println!("mutex is poisoned: {}", data.is_poisoned());
```

The panic occurred after pushing 4 but before any further operations. The data inside is actually valid (it contains `[1, 2, 3, 4]`), but the mutex is marked poisoned because the compiler cannot know whether the data is consistent.

To recover, use `PoisonError::into_inner()` to extract the guard.

```rust
match data.lock() {
    Ok(guard) => println!("data: {:?}", *guard),
    Err(poison_err) => {
        println!("recovering from poisoned mutex");
        let guard = poison_err.into_inner();
        println!("recovered data: {:?}", *guard); // [1, 2, 3, 4]
    }
}
```

The `into_inner()` call gives you the `MutexGuard`, which you can use normally. The lock is acquired and will be released when the guard is dropped.

If you want to clear the poisoned state so future callers do not see the error, use `clear_poison()`.

```rust
data.clear_poison();
assert!(!data.is_poisoned());

// Now lock() returns Ok again
let guard = data.lock().unwrap();
println!("data after clearing: {:?}", *guard);
```

This is useful after you have inspected and repaired the data, and want the mutex to behave normally again.

A common pattern is to always recover from poisoning, treating it as a non-issue for your specific data type.

```rust
use std::sync::Mutex;

fn get_value(m: &Mutex<i32>) -> i32 {
    // Recover from poisoning — an i32 counter is always valid
    let guard = m.lock().unwrap_or_else(|e| e.into_inner());
    *guard
}
```

The `unwrap_or_else` pattern attempts a normal lock, and if the mutex is poisoned, recovers the guard anyway. This is appropriate when the data type has no complex invariants that could be violated by a partial update.

## Common Pitfalls

1. **Always using `.unwrap()` on `lock()`** — In production code, this turns a poisoned mutex into a cascading panic. Use `unwrap_or_else(|e| e.into_inner())` for data types that are always valid, or match on the `Result` for data with complex invariants.
2. **Ignoring poisoning for complex data** — If your mutex protects a struct with invariants (e.g., a balanced tree or a transaction log), blindly recovering from poisoning can lead to data corruption. In these cases, propagate the error.

## Best Practices

1. **Use `unwrap_or_else` recovery for simple types** — Counters, flags, and append-only collections are always valid regardless of where a panic occurred. Use the recovery pattern to avoid unnecessary poison propagation.
2. **Consider `parking_lot::Mutex` when poisoning is unwanted** — If you never want to deal with poisoning, the `parking_lot` crate provides a `Mutex` that does not poison and returns the guard directly from `lock()`.

## Summary

- A mutex becomes poisoned when a thread panics while holding the `MutexGuard`.
- `PoisonError::into_inner()` recovers the guard and provides access to the data.
- `clear_poison()` (Rust 1.77) resets the poisoned state for future callers.
- Use `unwrap_or_else(|e| e.into_inner())` for data types that are always valid.
- `parking_lot::Mutex` avoids poisoning entirely — simpler API at the cost of no poison protection.

## Code Examples

**Recovering data from a poisoned mutex using unwrap_or_else and clearing the poison state**

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0i32));

    // Spawn a thread that panics while holding the lock
    let c = Arc::clone(&counter);
    let _ = thread::spawn(move || {
        let mut g = c.lock().unwrap();
        *g += 1;
        panic!("thread crashed!");
    }).join();

    // Recover from the poisoned mutex
    let value = counter.lock()
        .unwrap_or_else(|e| e.into_inner());
    println!("Counter value: {}", *value); // 1 — still valid

    // Clear poison for future callers
    counter.clear_poison();
    println!("Poisoned: {}", counter.is_poisoned()); // false
}
```


## Resources

- [std::sync::PoisonError](https://doc.rust-lang.org/std/sync/struct.PoisonError.html) — Standard library reference for PoisonError and recovery methods
- [Mutex::clear_poison](https://doc.rust-lang.org/std/sync/struct.Mutex.html#method.clear_poison) — Method to clear the poisoned state, stabilized in Rust 1.77

---

> 📘 *This lesson is part of the [Concurrent Rust](https://stanza.dev/courses/rust-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*