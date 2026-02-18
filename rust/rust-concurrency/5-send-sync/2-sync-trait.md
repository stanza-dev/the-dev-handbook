---
source_course: "rust-concurrency"
source_lesson: "rust-concurrency-sync-trait"
---

# Sync: Safe to Share References

`T` is `Sync` if `&T` is `Send` - meaning it's safe to access from multiple threads simultaneously.

```rust
use std::thread;

fn requires_sync<T: Sync>(t: &T) {
    thread::scope(|s| {
        s.spawn(|| println!("{:?}", t));
        s.spawn(|| println!("{:?}", t));
    });
}
```

## What's NOT Sync?

- **`Cell<T>`, `RefCell<T>`**: Interior mutability without synchronization
- **`Rc<T>`**: Non-atomic operations
- **`UnsafeCell<T>`**: The primitive for interior mutability

## The Relationship Table

| Type | Send | Sync | Why |
|------|------|------|-----|
| `i32` | ✓ | ✓ | Immutable sharing is fine |
| `String` | ✓ | ✓ | Immutable sharing is fine |
| `Rc<i32>` | ✗ | ✗ | Non-atomic count |
| `Arc<i32>` | ✓ | ✓ | Atomic count |
| `Cell<i32>` | ✓ | ✗ | Can mutate via &, not sync |
| `RefCell<i32>` | ✓ | ✗ | Runtime borrow check not sync |
| `Mutex<i32>` | ✓ | ✓ | Locking provides sync |
| `MutexGuard<i32>` | ✗ | ✓ | Must unlock on same thread |

## Key Insight: Mutex Makes Things Sync

```rust
use std::sync::Mutex;
use std::cell::RefCell;

// RefCell: !Sync
// But Mutex<RefCell<T>>: Sync!
// The mutex ensures only one thread accesses the RefCell at a time

let shared = Arc::new(Mutex::new(RefCell::new(0)));
```

## The Rule

- `T: Sync` ≡ `&T: Send`
- If you can safely share `&T` between threads, `T` is `Sync`

See [Send and Sync](https://doc.rust-lang.org/nomicon/send-and-sync.html).

## Code Examples

**Making types thread-safe**

```rust
use std::sync::Arc;
use std::cell::RefCell;

// This doesn't compile:
// let shared: Arc<RefCell<i32>> = Arc::new(RefCell::new(0));
// thread::spawn({ let s = Arc::clone(&shared); move || { ... } });
// Error: RefCell<i32> cannot be shared between threads safely

// Fix 1: Use Mutex for interior mutability
use std::sync::Mutex;
let shared = Arc::new(Mutex::new(0));

// Fix 2: Use atomic types
use std::sync::atomic::AtomicI32;
let shared = Arc::new(AtomicI32::new(0));

// Fix 3: Use parking_lot::RwLock for read-heavy
use parking_lot::RwLock;
let shared = Arc::new(RwLock::new(0));
```


## Resources

- [Send and Sync](https://doc.rust-lang.org/nomicon/send-and-sync.html) — Rustonomicon chapter on Send and Sync

---

> 📘 *This lesson is part of the [Concurrent Rust](https://stanza.dev/courses/rust-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*