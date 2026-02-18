---
source_course: "rust-concurrency"
source_lesson: "rust-concurrency-memory-ordering"
---

# Why Memory Ordering Matters

CPUs and compilers reorder instructions for performance. This can break assumptions in concurrent code.

## Ordering Levels

| Ordering | Guarantees |
|----------|------------|
| `Relaxed` | Only atomicity, no ordering |
| `Acquire` | Reads after this see writes before matching Release |
| `Release` | Writes before this visible after matching Acquire |
| `AcqRel` | Both Acquire and Release |
| `SeqCst` | Total ordering across all threads (safest, slowest) |

## The Happens-Before Relationship

```rust
use std::sync::atomic::{AtomicBool, AtomicUsize, Ordering};

static DATA: AtomicUsize = AtomicUsize::new(0);
static READY: AtomicBool = AtomicBool::new(false);

// Thread 1: Writer
DATA.store(42, Ordering::Relaxed);
READY.store(true, Ordering::Release);  // Release "publishes" DATA

// Thread 2: Reader
while !READY.load(Ordering::Acquire) {} // Acquire "sees" DATA
let data = DATA.load(Ordering::Relaxed);
assert_eq!(data, 42);  // Guaranteed!
```

## Common Patterns

### Relaxed: Counters
```rust
counter.fetch_add(1, Ordering::Relaxed);
// We only care that increments don't get lost
```

### Release-Acquire: Lock-like patterns
```rust
// Unlock: Release
locked.store(false, Ordering::Release);

// Lock: Acquire  
while locked.compare_exchange_weak(
    false, true,
    Ordering::Acquire, Ordering::Relaxed
).is_err() {}
```

### SeqCst: When in doubt
```rust
// Strongest guarantees, always correct
flag.store(true, Ordering::SeqCst);
```

See [Atomics](https://doc.rust-lang.org/nomicon/atomics.html).

## Code Examples

**Spinlock implementation**

```rust
use std::sync::atomic::{AtomicBool, Ordering};

// Simple spinlock
struct SpinLock {
    locked: AtomicBool,
}

impl SpinLock {
    const fn new() -> Self {
        SpinLock { locked: AtomicBool::new(false) }
    }
    
    fn lock(&self) {
        while self.locked
            .compare_exchange_weak(
                false,
                true,
                Ordering::Acquire,  // Acquire on success
                Ordering::Relaxed,  // Relaxed on failure
            )
            .is_err()
        {
            std::hint::spin_loop();
        }
    }
    
    fn unlock(&self) {
        self.locked.store(false, Ordering::Release);
    }
}
```


## Resources

- [Atomics in Rust](https://doc.rust-lang.org/nomicon/atomics.html) — Rustonomicon chapter on atomics

---

> 📘 *This lesson is part of the [Concurrent Rust](https://stanza.dev/courses/rust-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*