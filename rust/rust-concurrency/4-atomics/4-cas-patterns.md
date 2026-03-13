---
source_course: "rust-concurrency"
source_lesson: "rust-concurrency-cas-patterns"
---

# Compare-and-Swap Patterns

## Introduction

Compare-and-swap (CAS) is the fundamental building block of lock-free programming. It atomically checks whether a value equals an expected value and, if so, replaces it with a new value. When combined with a retry loop, CAS enables arbitrary atomic read-modify-write operations — from simple counters to complex lock-free data structures.

## Key Concepts

- **CAS Loop**: Read the current value, compute the desired new value, attempt the swap. If another thread changed the value between the read and the swap, retry with the updated value.
- **Spurious Failure**: `compare_exchange_weak` may fail even when the current value matches the expected value. This is cheaper on ARM/RISC-V architectures and harmless in retry loops.
- **ABA Problem**: If a value changes from A to B and back to A, CAS sees A and succeeds, even though the value was modified. This can cause bugs in pointer-based data structures.
- **Atomic Maximum/Minimum**: A CAS loop can implement any read-modify-write operation, including max and min, which have no dedicated atomic instruction.

## Real World Context

CAS loops power the internals of `Arc` (for reference count manipulation), lock-free queues (Michael-Scott queue), and concurrent hash maps. Any time you need to atomically update a value based on its current state without holding a lock, CAS is the tool. Understanding CAS patterns is essential for reading and contributing to high-performance Rust crates.

## Deep Dive

The basic CAS loop pattern reads the current value, computes the desired value, and attempts the swap.

```rust
use std::sync::atomic::{AtomicI64, Ordering};

fn atomic_max(atom: &AtomicI64, new_val: i64) -> i64 {
    let mut current = atom.load(Ordering::Relaxed);
    loop {
        let desired = current.max(new_val);
        if desired == current {
            return current; // already at or above new_val
        }
        match atom.compare_exchange_weak(
            current,
            desired,
            Ordering::AcqRel,
            Ordering::Acquire,
        ) {
            Ok(prev) => return prev,
            Err(actual) => current = actual,
        }
    }
}
```

This implements an atomic maximum: it updates the atomic value to `max(current, new_val)`. If another thread changes the value between the load and the CAS, the loop retries with the new current value.

The same pattern works for any function you want to apply atomically.

```rust
use std::sync::atomic::{AtomicU64, Ordering};

fn atomic_update<F>(atom: &AtomicU64, f: F) -> u64
where
    F: Fn(u64) -> u64,
{
    let mut current = atom.load(Ordering::Relaxed);
    loop {
        let new = f(current);
        match atom.compare_exchange_weak(
            current,
            new,
            Ordering::AcqRel,
            Ordering::Acquire,
        ) {
            Ok(prev) => return prev,
            Err(actual) => current = actual,
        }
    }
}

let val = AtomicU64::new(100);
atomic_update(&val, |v| v.saturating_sub(30)); // atomic subtract with floor at 0
assert_eq!(val.load(Ordering::Relaxed), 70);
```

This generic `atomic_update` function accepts any closure, making it a reusable building block for custom atomic operations.

A common real-world use is implementing a lock-free ID generator.

```rust
use std::sync::atomic::{AtomicU64, Ordering};

static NEXT_ID: AtomicU64 = AtomicU64::new(1);

fn generate_id() -> u64 {
    NEXT_ID.fetch_add(1, Ordering::Relaxed)
}

// Each call returns a unique ID: 1, 2, 3, ...
let id1 = generate_id(); // 1
let id2 = generate_id(); // 2
```

In this case, `fetch_add` is a built-in CAS operation and does not need an explicit loop. But for operations without a dedicated `fetch_*` method (like max, min, or conditional updates), the CAS loop is necessary.

## Common Pitfalls

1. **Using `compare_exchange` in loops** — The non-weak variant includes an internal retry on LL/SC architectures, making it slightly more expensive per iteration. Use `compare_exchange_weak` in retry loops.
2. **Ignoring the ABA problem in pointer CAS** — When using `AtomicPtr`, a value can change from A to B and back to A. If you free and reallocate memory between these changes, a CAS on the pointer succeeds but points to different data. Use epoch-based reclamation or tagged pointers to prevent this.

## Best Practices

1. **Use `fetch_*` methods when available** — `fetch_add`, `fetch_sub`, `fetch_or`, `fetch_and`, and `fetch_xor` are simpler and often faster than explicit CAS loops. Only write a CAS loop for operations without a dedicated method.
2. **Keep the computation between load and CAS minimal** — The longer you spend computing the new value, the more likely another thread will change the value, causing a retry. Move expensive work outside the CAS loop when possible.

## Summary

- CAS loops enable arbitrary atomic read-modify-write operations: load, compute, swap, retry on failure.
- Use `compare_exchange_weak` in loops for better performance on ARM/RISC-V.
- `fetch_*` methods are preferred when available — they are simpler and often hardware-optimized.
- Watch for the ABA problem in pointer-based CAS: use epoch reclamation or tagged pointers.
- Keep computation inside CAS loops minimal to reduce retry frequency.

## Code Examples

**Lock-free atomic maximum using a CAS loop — multiple threads compete to set the highest value**

```rust
use std::sync::atomic::{AtomicI64, Ordering};
use std::sync::Arc;
use std::thread;

/// Atomically update the value to the maximum of current and new_val.
fn atomic_max(atom: &AtomicI64, new_val: i64) -> i64 {
    let mut current = atom.load(Ordering::Relaxed);
    loop {
        let desired = current.max(new_val);
        if desired == current {
            return current;
        }
        match atom.compare_exchange_weak(
            current, desired,
            Ordering::AcqRel, Ordering::Acquire,
        ) {
            Ok(prev) => return prev,
            Err(actual) => current = actual,
        }
    }
}

fn main() {
    let max_val = Arc::new(AtomicI64::new(0));
    let handles: Vec<_> = [42, 17, 99, 3, 88].iter().map(|&v| {
        let max_val = Arc::clone(&max_val);
        thread::spawn(move || { atomic_max(&max_val, v); })
    }).collect();
    for h in handles { h.join().unwrap(); }
    println!("Max: {}", max_val.load(Ordering::Relaxed)); // 99
}
```


## Resources

- [std::sync::atomic::AtomicUsize::compare_exchange](https://doc.rust-lang.org/std/sync/atomic/struct.AtomicUsize.html#method.compare_exchange) — Standard library reference for compare_exchange (CAS)
- [Rust Atomics and Locks — Chapter 4](https://marabos.nl/atomics/building-spinlocks.html) — Building spinlocks and other lock-free structures with CAS

---

> 📘 *This lesson is part of the [Concurrent Rust](https://stanza.dev/courses/rust-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*