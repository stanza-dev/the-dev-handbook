---
source_course: "rust-concurrency"
source_lesson: "rust-concurrency-atomic-types"
---

# Lock-Free Operations

Atomics allow thread-safe access without locks using CPU instructions.

```rust
use std::sync::atomic::{AtomicUsize, Ordering};

static COUNTER: AtomicUsize = AtomicUsize::new(0);

fn increment() {
    COUNTER.fetch_add(1, Ordering::SeqCst);
}

fn get() -> usize {
    COUNTER.load(Ordering::SeqCst)
}
```

## Atomic Types Available

- `AtomicBool`
- `AtomicI8/16/32/64/size`
- `AtomicU8/16/32/64/size`
- `AtomicPtr<T>`

## Operations

```rust
let x = AtomicUsize::new(0);

// Load and store
let val = x.load(Ordering::Acquire);
x.store(42, Ordering::Release);

// Fetch and modify (returns OLD value)
let old = x.fetch_add(1, Ordering::SeqCst);
let old = x.fetch_sub(1, Ordering::SeqCst);
let old = x.fetch_or(0xFF, Ordering::SeqCst);
let old = x.fetch_and(0x0F, Ordering::SeqCst);

// Compare and swap
let result = x.compare_exchange(
    5,                    // Expected
    10,                   // New value
    Ordering::SeqCst,     // Success ordering
    Ordering::SeqCst,     // Failure ordering
);
// result: Ok(5) if swapped, Err(current) if not

// Weak version (may fail spuriously but faster in loops)
let result = x.compare_exchange_weak(5, 10, Ordering::SeqCst, Ordering::Relaxed);
```

## AtomicBool for Flags

```rust
use std::sync::atomic::AtomicBool;

static SHUTDOWN: AtomicBool = AtomicBool::new(false);

fn signal_shutdown() {
    SHUTDOWN.store(true, Ordering::Release);
}

fn should_shutdown() -> bool {
    SHUTDOWN.load(Ordering::Acquire)
}
```

See [Atomics](https://doc.rust-lang.org/std/sync/atomic/).

## Code Examples

**Atomic counter**

```rust
use std::sync::atomic::{AtomicUsize, Ordering};
use std::sync::Arc;
use std::thread;

// Lock-free counter
let counter = Arc::new(AtomicUsize::new(0));
let mut handles = vec![];

for _ in 0..10 {
    let counter = Arc::clone(&counter);
    handles.push(thread::spawn(move || {
        for _ in 0..1000 {
            counter.fetch_add(1, Ordering::Relaxed);
        }
    }));
}

for h in handles {
    h.join().unwrap();
}

println!("Count: {}", counter.load(Ordering::Relaxed)); // 10000
```


---

> 📘 *This lesson is part of the [Concurrent Rust](https://stanza.dev/courses/rust-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*