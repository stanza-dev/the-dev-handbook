---
source_course: "rust-concurrency"
source_lesson: "rust-concurrency-atomic-types"
---

# Atomic Types

## Introduction

Atomic types provide lock-free, thread-safe operations on primitive values. Unlike `Mutex`, which blocks other threads while one thread holds the lock, atomic operations complete in a single CPU instruction — no thread ever waits for a lock. This makes them the foundation of high-performance concurrent data structures, counters, flags, and synchronization primitives.

## Key Concepts

- **Atomic Types**: `AtomicBool`, `AtomicI8`/`AtomicI16`/`AtomicI32`/`AtomicI64`/`AtomicIsize`, `AtomicU8`/`AtomicU16`/`AtomicU32`/`AtomicU64`/`AtomicUsize`, and `AtomicPtr<T>`. Each wraps a primitive type and provides atomic operations.
- **load / store**: Read or write the value atomically. Every access to an atomic must specify a memory ordering.
- **fetch_add / fetch_sub / fetch_and / fetch_or / fetch_xor**: Read-modify-write operations that execute atomically. They return the *previous* value before the operation was applied.
- **compare_exchange**: The compare-and-swap (CAS) operation. Atomically compares the current value to an expected value; if they match, it writes a new value and returns `Ok(previous)`. If they do not match, it returns `Err(current_value)` without modifying anything.
- **Static Atomics**: Atomic types implement `const fn new()`, so they can be used directly in `static` declarations without `Arc` or lazy initialization.

## Real World Context

Atomics are everywhere in systems programming. Reference counts in `Arc` use `AtomicUsize` internally. Lock-free queues, hazard pointers, and epoch-based reclamation all build on atomic operations. Simple use cases include global request counters, feature flags that can be toggled at runtime, and shutdown signals. When you need a shared counter or flag and a full `Mutex` feels like overkill, atomics are the right tool.

## Deep Dive

The simplest atomic pattern is a shared counter. Unlike `Arc<Mutex<usize>>`, an `AtomicUsize` requires no lock acquisition — the CPU handles the atomicity directly.

```rust
use std::sync::atomic::{AtomicUsize, Ordering};
use std::sync::Arc;
use std::thread;

let counter = Arc::new(AtomicUsize::new(0));

let handles: Vec<_> = (0..10)
    .map(|_| {
        let counter = Arc::clone(&counter);
        thread::spawn(move || {
            for _ in 0..1000 {
                counter.fetch_add(1, Ordering::Relaxed);
            }
        })
    })
    .collect();

for h in handles {
    h.join().unwrap();
}

println!("count: {}", counter.load(Ordering::Relaxed)); // 10000
```

Atomic types can be declared as `static` variables, which avoids the need for `Arc` entirely. This is the most common pattern for global counters and flags.

```rust
use std::sync::atomic::{AtomicBool, AtomicU64, Ordering};

static RUNNING: AtomicBool = AtomicBool::new(true);
static REQUEST_COUNT: AtomicU64 = AtomicU64::new(0);

fn handle_request() {
    REQUEST_COUNT.fetch_add(1, Ordering::Relaxed);
    // ... handle the request ...
}

fn shutdown() {
    RUNNING.store(false, Ordering::Release);
}

fn is_running() -> bool {
    RUNNING.load(Ordering::Acquire)
}
```

The `store` and `load` methods atomically write and read the value. They are the simplest atomic operations.

```rust
use std::sync::atomic::{AtomicI32, Ordering};

let value = AtomicI32::new(10);

value.store(42, Ordering::SeqCst);
let current = value.load(Ordering::SeqCst);
println!("current: {current}"); // 42
```

The `fetch_*` family of operations read the current value, apply an operation, and store the result — all atomically. Critically, they return the value *before* the operation.

```rust
use std::sync::atomic::{AtomicI32, Ordering};

let value = AtomicI32::new(10);

// fetch_add: returns old value, then adds
let old = value.fetch_add(5, Ordering::SeqCst);
assert_eq!(old, 10);  // returned the OLD value
assert_eq!(value.load(Ordering::SeqCst), 15); // now 15

// fetch_sub: returns old value, then subtracts
let old = value.fetch_sub(3, Ordering::SeqCst);
assert_eq!(old, 15);
assert_eq!(value.load(Ordering::SeqCst), 12);

// Bitwise operations work too
let flags = AtomicI32::new(0b1100);
let old = flags.fetch_or(0b0011, Ordering::SeqCst);
assert_eq!(old, 0b1100);
assert_eq!(flags.load(Ordering::SeqCst), 0b1111);
```

`compare_exchange` is the most powerful atomic operation — the foundation of lock-free algorithms. It compares the current value to an expected value and, only if they match, replaces it with a new value.

```rust
use std::sync::atomic::{AtomicI32, Ordering};

let value = AtomicI32::new(5);

// Success: current value (5) matches expected (5)
match value.compare_exchange(5, 10, Ordering::SeqCst, Ordering::SeqCst) {
    Ok(prev) => println!("swapped: {prev} -> 10"),  // prev is 5
    Err(actual) => println!("failed, current is {actual}"),
}

// Failure: current value (10) does not match expected (5)
match value.compare_exchange(5, 20, Ordering::SeqCst, Ordering::SeqCst) {
    Ok(prev) => println!("swapped: {prev} -> 20"),
    Err(actual) => println!("failed, current is {actual}"),  // actual is 10
}
```

`AtomicPtr<T>` stores a raw pointer atomically. It is the building block for lock-free data structures like stacks and queues.

```rust
use std::sync::atomic::{AtomicPtr, Ordering};
use std::ptr;

let data = Box::new(42);
let atomic_ptr = AtomicPtr::new(Box::into_raw(data));

// Atomically load the pointer
let ptr = atomic_ptr.load(Ordering::Acquire);
unsafe {
    println!("value: {}", *ptr);
}

// Clean up
unsafe {
    drop(Box::from_raw(atomic_ptr.load(Ordering::Acquire)));
}
```

## Common Pitfalls

1. **Using the wrong memory ordering** — `Ordering::Relaxed` provides no synchronization guarantees beyond atomicity. If you are using an atomic to communicate data between threads (not just count), you likely need `Acquire`/`Release` or `SeqCst`.
2. **Assuming `fetch_add` returns the new value** — All `fetch_*` operations return the *previous* value. If you need the value after the operation, compute it yourself: `let new = counter.fetch_add(1, Ordering::Relaxed) + 1;`.

## Best Practices

1. **Use `static` atomics for global counters and flags** — No `Arc` needed. Atomic types have a `const fn new()` constructor that works in static context.
2. **Start with `SeqCst` and optimize later** — `SeqCst` is the safest ordering. Only weaken to `Acquire`/`Release` or `Relaxed` after you understand the ordering requirements and have verified correctness.

## Summary

- Atomic types (`AtomicBool`, `AtomicI*`, `AtomicU*`, `AtomicPtr`) provide lock-free thread-safe operations on primitive values.
- `load`/`store` atomically read/write; `fetch_add`/`fetch_sub`/`fetch_or`/`fetch_xor` perform atomic read-modify-write and return the previous value.
- `compare_exchange` is the CAS primitive underlying lock-free algorithms.
- Static atomics avoid `Arc` overhead and are the standard pattern for global counters and flags.

## Code Examples

**Lock-free global counter using a static AtomicU64 shared across 8 threads**

```rust
use std::sync::atomic::{AtomicU64, Ordering};
use std::sync::Arc;
use std::thread;

static GLOBAL_COUNTER: AtomicU64 = AtomicU64::new(0);

fn main() {
    let handles: Vec<_> = (0..8)
        .map(|_| {
            thread::spawn(|| {
                for _ in 0..1000 {
                    GLOBAL_COUNTER.fetch_add(1, Ordering::Relaxed);
                }
            })
        })
        .collect();

    for h in handles {
        h.join().unwrap();
    }

    println!("Total: {}", GLOBAL_COUNTER.load(Ordering::Relaxed));
    // Always prints 8000
}
```


## Resources

- [std::sync::atomic module](https://doc.rust-lang.org/std/sync/atomic/index.html) — Standard library reference for all atomic types and operations
- [Rust Atomics and Locks (book)](https://marabos.nl/atomics/) — Mara Bos's comprehensive book on atomics, locks, and memory ordering in Rust

---

> 📘 *This lesson is part of the [Concurrent Rust](https://stanza.dev/courses/rust-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*