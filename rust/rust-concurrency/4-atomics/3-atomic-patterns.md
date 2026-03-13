---
source_course: "rust-concurrency"
source_lesson: "rust-concurrency-atomic-patterns"
---

# Atomic Patterns & Fences

## Introduction

Beyond simple counters and flags, atomic operations enable sophisticated lock-free algorithms. The compare-and-swap (CAS) loop is the fundamental building block: it lets you atomically read a value, compute a new value, and write it back only if no other thread changed it in between. Fences provide ordering guarantees without being tied to a specific atomic variable. Together, these tools power the lock-free data structures used in high-performance systems.

## Key Concepts

- **CAS Loop**: A retry loop built around `compare_exchange`. Read the current value, compute the desired new value, attempt the swap. If another thread changed the value between the read and the swap, the CAS fails and you retry with the updated value.
- **compare_exchange vs compare_exchange_weak**: `compare_exchange` guarantees it only fails when the current value differs from expected. `compare_exchange_weak` may also fail "spuriously" (even when the values match) on some architectures (notably ARM). Use `_weak` in CAS loops where you will retry anyway — it can be faster because it avoids an internal retry on LL/SC architectures.
- **AtomicPtr<T>**: Stores a raw pointer atomically. It is the building block for lock-free linked lists, stacks, and queues. All pointer operations (`load`, `store`, `compare_exchange`) are atomic.
- **Fence**: `std::sync::atomic::fence(Ordering)` establishes ordering constraints without being tied to a specific atomic variable. A `Release` fence followed by a `Relaxed` store is equivalent to a `Release` store, but fences can be more efficient when you need to synchronize multiple atomic variables at once.
- **compiler_fence**: `std::sync::atomic::compiler_fence(Ordering)` prevents the *compiler* from reordering operations across the fence but does not emit a CPU memory barrier. Used in signal handlers and when interfacing with hardware that provides its own ordering guarantees.

## Real World Context

Lock-free stacks (Treiber stack) and queues (Michael-Scott queue) use CAS loops on `AtomicPtr` to push and pop elements without locks. The `Arc` type in the standard library uses CAS on its reference count to handle concurrent cloning and dropping. Sequence locks use fences to ensure readers see a consistent snapshot. Epoch-based reclamation (used by `crossbeam-epoch`) uses atomics and fences to safely defer deallocation of shared data until all readers have finished.

## Deep Dive

### CAS Loops

The canonical CAS loop pattern: load the current value, compute the desired value, attempt to swap. If the swap fails, use the returned current value and retry.

```rust
use std::sync::atomic::{AtomicU64, Ordering};

fn atomic_multiply(value: &AtomicU64, multiplier: u64) -> u64 {
    let mut current = value.load(Ordering::Relaxed);
    loop {
        let new = current * multiplier;
        match value.compare_exchange_weak(
            current,
            new,
            Ordering::AcqRel,
            Ordering::Acquire,
        ) {
            Ok(prev) => return prev, // success: return the old value
            Err(actual) => current = actual, // retry with updated value
        }
    }
}

let val = AtomicU64::new(10);
let old = atomic_multiply(&val, 3);
assert_eq!(old, 10);
assert_eq!(val.load(Ordering::Relaxed), 30);
```

### compare_exchange vs compare_exchange_weak

`compare_exchange` is guaranteed to succeed when the current value matches the expected value. `compare_exchange_weak` may fail spuriously, which is cheaper on ARM and other LL/SC (Load-Linked / Store-Conditional) architectures because the CPU does not need an internal retry loop.

```rust
use std::sync::atomic::{AtomicI32, Ordering};

let value = AtomicI32::new(0);

// Use compare_exchange when you need a single attempt
// (e.g., trying to acquire a lock)
match value.compare_exchange(0, 1, Ordering::Acquire, Ordering::Relaxed) {
    Ok(_) => println!("lock acquired"),
    Err(_) => println!("lock was already held"),
}

// Use compare_exchange_weak in CAS loops (cheaper on ARM)
let mut current = value.load(Ordering::Relaxed);
loop {
    match value.compare_exchange_weak(
        current,
        current + 10,
        Ordering::AcqRel,
        Ordering::Relaxed,
    ) {
        Ok(_) => break,
        Err(actual) => current = actual,
    }
}
```

**Rule of thumb**: Use `compare_exchange_weak` in loops, `compare_exchange` for one-shot attempts.

### AtomicPtr<T> for Lock-Free Structures

`AtomicPtr` enables lock-free data structures by allowing atomic pointer swaps. Here is a simplified lock-free stack (Treiber stack).

```rust
use std::sync::atomic::{AtomicPtr, Ordering};
use std::ptr;

struct Node<T> {
    value: T,
    next: *mut Node<T>,
}

struct LockFreeStack<T> {
    head: AtomicPtr<Node<T>>,
}

impl<T> LockFreeStack<T> {
    fn new() -> Self {
        Self {
            head: AtomicPtr::new(ptr::null_mut()),
        }
    }

    fn push(&self, value: T) {
        let new_node = Box::into_raw(Box::new(Node {
            value,
            next: ptr::null_mut(),
        }));

        loop {
            let old_head = self.head.load(Ordering::Acquire);
            unsafe { (*new_node).next = old_head; }

            match self.head.compare_exchange_weak(
                old_head,
                new_node,
                Ordering::AcqRel,
                Ordering::Acquire,
            ) {
                Ok(_) => break,
                Err(_) => continue, // another thread pushed, retry
            }
        }
    }

    fn pop(&self) -> Option<T> {
        loop {
            let old_head = self.head.load(Ordering::Acquire);
            if old_head.is_null() {
                return None;
            }

            let next = unsafe { (*old_head).next };

            match self.head.compare_exchange_weak(
                old_head,
                next,
                Ordering::AcqRel,
                Ordering::Acquire,
            ) {
                Ok(_) => {
                    let node = unsafe { Box::from_raw(old_head) };
                    return Some(node.value);
                }
                Err(_) => continue,
            }
        }
    }
}
```

Note: This simplified stack has the ABA problem. Production implementations use epoch-based reclamation (e.g., `crossbeam-epoch`) or hazard pointers to safely reclaim memory.

### Standalone Fences

`std::sync::atomic::fence()` provides ordering guarantees that are not tied to a specific atomic variable. A fence applies to *all* atomic operations before or after it.

```rust
use std::sync::atomic::{self, AtomicBool, AtomicU64, Ordering};

static A: AtomicU64 = AtomicU64::new(0);
static B: AtomicU64 = AtomicU64::new(0);
static FLAG: AtomicBool = AtomicBool::new(false);

// Writer: write multiple values, then Release fence + Relaxed store
A.store(1, Ordering::Relaxed);
B.store(2, Ordering::Relaxed);
atomic::fence(Ordering::Release); // all previous writes are now "published"
FLAG.store(true, Ordering::Relaxed);

// Reader: Relaxed load + Acquire fence, then read data
if FLAG.load(Ordering::Relaxed) {
    atomic::fence(Ordering::Acquire); // all subsequent reads see published data
    assert_eq!(A.load(Ordering::Relaxed), 1);
    assert_eq!(B.load(Ordering::Relaxed), 2);
}
```

Fences are useful when you need to synchronize access to multiple atomic variables simultaneously. Without a fence, you would need `Release`/`Acquire` on each individual store/load.

### compiler_fence

`compiler_fence` prevents the compiler from reordering operations but does not emit a hardware memory barrier. This is useful in signal handlers (which run on the same thread as the interrupted code, so CPU ordering is already guaranteed) and when interfacing with memory-mapped hardware.

```rust
use std::sync::atomic::{compiler_fence, Ordering};

// Prevent the compiler from reordering these writes
unsafe {
    write_to_hardware_register(0x01);
    compiler_fence(Ordering::SeqCst);
    write_to_hardware_register(0x02); // guaranteed to happen after 0x01
}
```

### High-Level Patterns

**Sequence Lock**: A reader-writer synchronization mechanism where writers increment a sequence counter before and after writing. Readers check the counter before and after reading — if it changed, the read was inconsistent and must be retried.

```rust
use std::sync::atomic::{AtomicUsize, Ordering, fence};

struct SeqLock<T: Copy> {
    seq: AtomicUsize,
    data: std::cell::UnsafeCell<T>,
}

unsafe impl<T: Copy + Send> Sync for SeqLock<T> {}

impl<T: Copy> SeqLock<T> {
    fn read(&self) -> T {
        loop {
            let s1 = self.seq.load(Ordering::Acquire);
            if s1 & 1 != 0 {
                // Odd means a write is in progress, spin
                std::hint::spin_loop();
                continue;
            }

            let data = unsafe { *self.data.get() };
            fence(Ordering::Acquire);

            let s2 = self.seq.load(Ordering::Relaxed);
            if s1 == s2 {
                return data; // consistent read
            }
            // Sequence changed — a write occurred during our read, retry
        }
    }
}
```

**Epoch-Based Reclamation** (high-level overview): Threads announce they are in a "critical section" by incrementing a per-thread epoch counter. When a writer removes a node, it defers deallocation until all threads have advanced past the epoch in which the node was removed. This avoids the ABA problem and use-after-free without requiring garbage collection. The `crossbeam-epoch` crate provides a production-ready implementation.

## Common Pitfalls

1. **The ABA problem with CAS** — If a value changes from A to B and back to A, a CAS operation sees A and succeeds, even though the value was modified. In pointer-based structures, this can cause use-after-free. Use epoch-based reclamation or tagged pointers to prevent ABA.
2. **Using `compare_exchange` in tight loops** — On ARM/RISC-V (LL/SC architectures), `compare_exchange` includes an internal retry that makes it more expensive in loops. Use `compare_exchange_weak` in CAS loops for better performance on these platforms.

## Best Practices

1. **Use `compare_exchange_weak` in CAS loops** — The spurious failures are harmless in a loop (you retry anyway), and `_weak` is faster on LL/SC architectures.
2. **Use `crossbeam-epoch` for lock-free data structures** — Manually managing memory in lock-free structures is extremely error-prone. Epoch-based reclamation handles the hard parts (deferred deallocation, ABA prevention) safely.

## Summary

- CAS loops are the building block of lock-free algorithms: load, compute, compare-and-swap, retry on failure.
- `compare_exchange_weak` is cheaper in loops; `compare_exchange` for one-shot attempts.
- `AtomicPtr<T>` enables lock-free data structures like stacks and queues.
- `fence()` provides ordering guarantees across multiple atomic variables; `compiler_fence()` prevents compiler reordering only.
- Sequence locks and epoch-based reclamation are advanced patterns built on atomic operations and fences.

## Code Examples

**Generic CAS loop pattern: atomically apply an arbitrary function to an atomic value**

```rust
use std::sync::atomic::{AtomicU64, Ordering};
use std::sync::Arc;
use std::thread;

/// Atomically update a value using a CAS loop.
/// Applies `f` to the current value until the swap succeeds.
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

fn main() {
    let value = Arc::new(AtomicU64::new(1));

    let handles: Vec<_> = (0..4)
        .map(|_| {
            let value = Arc::clone(&value);
            thread::spawn(move || {
                atomic_update(&value, |v| v * 2);
            })
        })
        .collect();

    for h in handles {
        h.join().unwrap();
    }

    // After 4 doublings: 1 * 2 * 2 * 2 * 2 = 16
    println!("Final: {}", value.load(Ordering::Relaxed));
}
```


## Resources

- [std::sync::atomic::fence](https://doc.rust-lang.org/std/sync/atomic/fn.fence.html) — Standard library reference for standalone memory fences
- [std::sync::atomic::compiler_fence](https://doc.rust-lang.org/std/sync/atomic/fn.compiler_fence.html) — Compiler-only fence for signal handlers and hardware I/O
- [Rust Atomics and Locks — Chapter 4-6](https://marabos.nl/atomics/) — Advanced atomic patterns, spinlocks, and lock-free data structures
- [crossbeam-epoch documentation](https://docs.rs/crossbeam-epoch/) — Epoch-based memory reclamation for lock-free data structures

---

> 📘 *This lesson is part of the [Concurrent Rust](https://stanza.dev/courses/rust-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*