---
source_course: "rust-concurrency"
source_lesson: "rust-concurrency-memory-ordering"
---

# Memory Ordering

## Introduction

Every atomic operation in Rust requires a `std::sync::atomic::Ordering` parameter. This is not just bureaucracy — it controls how the CPU and compiler are allowed to reorder memory operations around the atomic access. Choosing the right ordering is the difference between correct concurrent code and subtle, intermittent bugs that only appear under high load on specific hardware.

## Key Concepts

- **Relaxed**: No ordering constraints beyond atomicity. The operation is atomic, but surrounding reads and writes can be reordered freely. Suitable only for independent counters where no other data depends on the atomic value.
- **Acquire**: Used on loads. Prevents any reads or writes *after* the load from being reordered *before* it. "I am acquiring access to shared data — make sure I see everything that was published before this value."
- **Release**: Used on stores. Prevents any reads or writes *before* the store from being reordered *after* it. "I am releasing (publishing) data — make sure everything I wrote is visible to whoever acquires this value."
- **AcqRel** (Acquire + Release): Used on read-modify-write operations (`fetch_add`, `compare_exchange`, etc.) that need both acquire and release semantics in a single operation.
- **SeqCst** (Sequentially Consistent): The strongest ordering. All `SeqCst` operations across all threads appear to execute in a single global order that every thread agrees on. This is the easiest to reason about but the most expensive on some architectures.
- **Happens-Before**: A formal relationship between operations. If operation A happens-before operation B, then B is guaranteed to see all memory effects of A. Acquire/Release pairs establish happens-before relationships.

## Real World Context

Most developers will use `SeqCst` or `Acquire`/`Release` pairs. `SeqCst` is the safe default when you are unsure — it prevents all surprises. `Acquire`/`Release` pairs are the workhorse of lock-free programming: the writer uses `Release` to publish data, and the reader uses `Acquire` to subscribe to it. `Relaxed` is reserved for truly independent statistics counters where no other data depends on the counter value. High-performance lock-free data structures in crates like `crossbeam` carefully use `AcqRel` on CAS operations.

## Deep Dive

### Relaxed: Atomicity Only

`Relaxed` guarantees that the operation is atomic (no torn reads or writes), but nothing else. Other threads may see the updates in a different order than they were performed.

```rust
use std::sync::atomic::{AtomicU64, Ordering};

static HIT_COUNT: AtomicU64 = AtomicU64::new(0);

fn record_hit() {
    // Relaxed is fine here: we only care about the final count,
    // not about ordering relative to other memory operations.
    HIT_COUNT.fetch_add(1, Ordering::Relaxed);
}

fn get_hits() -> u64 {
    HIT_COUNT.load(Ordering::Relaxed)
}
```

`Relaxed` is sufficient here because the hit count is independent of any other shared data. No other memory operation needs to be ordered relative to the counter increment.

### Acquire / Release: The Publishing Pattern

The Acquire-Release pair is the fundamental pattern for safely sharing data between threads. The writer stores data and then does a `Release` store on a flag. The reader does an `Acquire` load on the flag and then reads the data. The Release-Acquire pair guarantees that the reader sees all writes made by the writer before the flag was set.

```rust
use std::sync::atomic::{AtomicBool, Ordering};
use std::thread;

static DATA: AtomicBool = AtomicBool::new(false);
static mut VALUE: u64 = 0;

let writer = thread::spawn(|| {
    unsafe { VALUE = 42; } // write the data
    DATA.store(true, Ordering::Release); // publish: "data is ready"
});

let reader = thread::spawn(|| {
    while !DATA.load(Ordering::Acquire) {} // subscribe: wait for data
    // Acquire guarantees we see VALUE = 42
    unsafe { assert_eq!(VALUE, 42); }
});

writer.join().unwrap();
reader.join().unwrap();
```

A safer version of this pattern uses `AtomicPtr` instead of `static mut`.

```rust
use std::sync::atomic::{AtomicPtr, Ordering};
use std::ptr;
use std::thread;

let atomic_ptr = AtomicPtr::new(ptr::null_mut::<String>());
let atomic_ref = &atomic_ptr;

thread::scope(|s| {
    // Writer: create data, then publish the pointer with Release
    s.spawn(|| {
        let data = Box::into_raw(Box::new(String::from("hello")));
        atomic_ref.store(data, Ordering::Release);
    });

    // Reader: Acquire the pointer, then safely read the data
    s.spawn(|| {
        loop {
            let ptr = atomic_ref.load(Ordering::Acquire);
            if !ptr.is_null() {
                unsafe { println!("read: {}", &*ptr); }
                break;
            }
            std::hint::spin_loop();
        }
    });
});

// Clean up
let ptr = atomic_ptr.load(Ordering::Acquire);
if !ptr.is_null() {
    unsafe { drop(Box::from_raw(ptr)); }
}
```

This version avoids `static mut` entirely by publishing and subscribing through an `AtomicPtr`. The `Release` store guarantees that the heap-allocated string is fully written before the pointer becomes visible, and the `Acquire` load guarantees that the reader sees the complete string.

### AcqRel: For Read-Modify-Write

`AcqRel` combines `Acquire` and `Release` in a single read-modify-write operation. The "read" part has `Acquire` semantics and the "write" part has `Release` semantics.

```rust
use std::sync::atomic::{AtomicUsize, Ordering};

static LOCK_STATE: AtomicUsize = AtomicUsize::new(0);

// Spinlock acquire: read the current state (Acquire) and write locked (Release)
while LOCK_STATE.compare_exchange(
    0,    // expected: unlocked
    1,    // new: locked
    Ordering::AcqRel,   // success: acquire + release
    Ordering::Acquire,  // failure: just acquire (read current state)
).is_err() {
    std::hint::spin_loop();
}

// Critical section — protected by the lock

// Spinlock release: store unlocked with Release
LOCK_STATE.store(0, Ordering::Release);
```

In this spinlock, `AcqRel` on the CAS serves double duty: the `Acquire` side ensures the thread sees all prior writes protected by the lock, and the `Release` side ensures that when the lock is transferred, the next acquirer sees all writes made in this critical section.

### SeqCst: Total Order

`SeqCst` establishes a single total order of all `SeqCst` operations. Every thread agrees on the order in which `SeqCst` operations happened. This is the strongest guarantee and the easiest to reason about.

```rust
use std::sync::atomic::{AtomicBool, AtomicI32, Ordering};
use std::thread;

static X: AtomicBool = AtomicBool::new(false);
static Y: AtomicBool = AtomicBool::new(false);
static Z: AtomicI32 = AtomicI32::new(0);

let a = thread::spawn(|| {
    X.store(true, Ordering::SeqCst);
});

let b = thread::spawn(|| {
    Y.store(true, Ordering::SeqCst);
});

let c = thread::spawn(|| {
    while !X.load(Ordering::SeqCst) {}
    if Y.load(Ordering::SeqCst) {
        Z.fetch_add(1, Ordering::SeqCst);
    }
});

let d = thread::spawn(|| {
    while !Y.load(Ordering::SeqCst) {}
    if X.load(Ordering::SeqCst) {
        Z.fetch_add(1, Ordering::SeqCst);
    }
});

a.join().unwrap();
b.join().unwrap();
c.join().unwrap();
d.join().unwrap();

// With SeqCst, Z is guaranteed to be at least 1.
// With weaker orderings, Z could be 0 on some architectures.
assert!(Z.load(Ordering::SeqCst) >= 1);
```

With `SeqCst`, all threads agree on a single total order of X and Y stores. This means at least one of threads C or D must see both X and Y as true, guaranteeing Z is at least 1. With weaker orderings, threads C and D could disagree on the order of the stores, potentially seeing Z remain 0.

### Ordering Comparison

| Ordering | Guarantees | Use Case |
|----------|-----------|----------|
| `Relaxed` | Atomicity only | Independent counters, statistics |
| `Acquire` | No reordering of later ops before this load | Reader side of publish pattern |
| `Release` | No reordering of earlier ops after this store | Writer side of publish pattern |
| `AcqRel` | Acquire on read + Release on write | CAS in lock-free structures |
| `SeqCst` | Total global order across all threads | Default safe choice, multi-variable protocols |

## Common Pitfalls

1. **Using `Relaxed` when data depends on the atomic** — If you set a flag with `Relaxed` and another thread reads data based on that flag, the data may not be visible yet. Use `Release`/`Acquire` to establish a happens-before relationship.
2. **Mismatching orderings in a pair** — A `Release` store must be paired with an `Acquire` load on the same atomic to establish happens-before. Using `Release` on both sides or `Acquire` on both sides does not work.

## Best Practices

1. **Default to `SeqCst` until you need performance** — `SeqCst` is correct in all situations. Only weaken to `Acquire`/`Release` or `Relaxed` after profiling shows the ordering is a bottleneck and you fully understand the implications.
2. **Use `Acquire`/`Release` pairs for the publish pattern** — When one thread writes data and then sets a flag, and another thread reads the flag and then reads the data, `Release` on the flag store and `Acquire` on the flag load is the correct and efficient choice.

## Summary

- `Relaxed` provides atomicity only — no ordering guarantees. Use for independent counters.
- `Acquire`/`Release` pairs establish happens-before relationships for the publish-subscribe pattern.
- `AcqRel` combines both in a single read-modify-write operation.
- `SeqCst` provides a total global order — safest but most expensive.
- Always pair `Release` stores with `Acquire` loads on the same atomic variable.

## Code Examples

**Release-Acquire pattern: writer publishes data with Release, reader subscribes with Acquire**

```rust
use std::sync::atomic::{AtomicBool, Ordering};
use std::thread;

static READY: AtomicBool = AtomicBool::new(false);
static mut DATA: u64 = 0;

fn main() {
    thread::scope(|s| {
        // Writer: store data, then Release the flag
        s.spawn(|| {
            unsafe { DATA = 42; }
            READY.store(true, Ordering::Release);
        });

        // Reader: Acquire the flag, then read data
        s.spawn(|| {
            while !READY.load(Ordering::Acquire) {
                std::hint::spin_loop();
            }
            // Acquire guarantees DATA = 42 is visible
            unsafe {
                assert_eq!(DATA, 42);
                println!("Read DATA = {}", DATA);
            }
        });
    });
}
```


## Resources

- [std::sync::atomic::Ordering](https://doc.rust-lang.org/std/sync/atomic/enum.Ordering.html) — Standard library reference for all memory ordering variants
- [Rust Atomics and Locks — Chapter 3: Memory Ordering](https://marabos.nl/atomics/memory-ordering.html) — In-depth treatment of memory ordering in Rust by Mara Bos
- [The Rustonomicon — Atomics](https://doc.rust-lang.org/nomicon/atomics.html) — Unsafe Rust guide covering atomic operations and memory ordering

---

> 📘 *This lesson is part of the [Concurrent Rust](https://stanza.dev/courses/rust-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*