---
source_course: "rust-concurrency"
source_lesson: "rust-concurrency-interior-mutability-threads"
---

# Interior Mutability & Thread Safety

## Introduction

Interior mutability — the ability to mutate data through a shared reference (`&self`) — is fundamental to Rust's concurrency story. Types like `Cell`, `RefCell`, `Mutex`, and `RwLock` all provide interior mutability, but with very different thread safety characteristics. Understanding which interior mutability types are safe to share across threads and why is key to choosing the right tool for concurrent code.

## Key Concepts

- **UnsafeCell<T>**: The primitive building block for all interior mutability in Rust. It is the only way to obtain a `&mut T` from a `&T`. It is `!Sync`, meaning types built on it must provide their own synchronization to be thread-safe.
- **Cell<T> and RefCell<T>**: Single-threaded interior mutability. Both are `!Sync` because they provide unsynchronized mutation through `&self`. `Cell` uses copy semantics; `RefCell` uses runtime borrow checking.
- **Mutex<T> and RwLock<T>**: Thread-safe interior mutability. They wrap `UnsafeCell` but add synchronization (locks), making them `Sync`. They are the standard way to achieve mutable shared state across threads.
- **Atomic types**: Lock-free interior mutability. `AtomicBool`, `AtomicUsize`, etc. provide synchronized mutation through hardware atomic instructions. They are both `Send` and `Sync`.

## Real World Context

A common mistake is trying to use `Arc<RefCell<T>>` for shared mutable state across threads. This fails to compile because `RefCell` is `!Sync`. The correct pattern is `Arc<Mutex<T>>` or `Arc<RwLock<T>>`. Understanding the interior mutability hierarchy helps you immediately know which wrapper to reach for: `Cell`/`RefCell` for single-threaded code, `Mutex`/`RwLock` for multi-threaded code, and atomics for lock-free concurrent access.

## Deep Dive

All interior mutability traces back to `UnsafeCell`, which is `!Sync` by design.

```rust
use std::cell::UnsafeCell;

let cell = UnsafeCell::new(42);

// UnsafeCell provides raw access — you must ensure safety yourself
let ptr: *mut i32 = cell.get();
unsafe {
    *ptr = 100;
    println!("value: {}", *ptr);
}
```

`UnsafeCell` is intentionally `!Sync` because it provides no synchronization. Any type that wraps it must either remain `!Sync` (like `Cell` and `RefCell`) or add its own synchronization (like `Mutex`).

`Cell` and `RefCell` provide ergonomic APIs on top of `UnsafeCell` for single-threaded use.

```rust
use std::cell::{Cell, RefCell};

let counter = Cell::new(0);
counter.set(counter.get() + 1); // mutation through &Cell

let data = RefCell::new(vec![1, 2, 3]);
data.borrow_mut().push(4); // runtime-checked mutable borrow through &RefCell
```

Both `Cell` and `RefCell` are `!Sync`, so the compiler prevents sharing them between threads. This is enforced at compile time with zero runtime cost.

`Mutex` and `RwLock` add synchronization, making interior mutability thread-safe.

```rust
use std::sync::{Arc, Mutex, RwLock};
use std::thread;

// Mutex: exclusive access for any operation
let shared = Arc::new(Mutex::new(vec![1, 2, 3]));
thread::scope(|s| {
    s.spawn(|| shared.lock().unwrap().push(4));
    s.spawn(|| shared.lock().unwrap().push(5));
});
println!("{:?}", shared.lock().unwrap());

// RwLock: multiple readers OR one writer
let config = Arc::new(RwLock::new(String::from("v1")));
thread::scope(|s| {
    // Multiple concurrent readers
    s.spawn(|| println!("reader: {}", config.read().unwrap()));
    s.spawn(|| println!("reader: {}", config.read().unwrap()));
    // Writer gets exclusive access
    s.spawn(|| *config.write().unwrap() = String::from("v2"));
});
```

The key insight is that `Mutex` and `RwLock` are `Sync` even though they contain `UnsafeCell` internally, because the lock mechanism ensures only one thread accesses the inner data at a time (or multiple readers in the `RwLock` case).

The following table summarizes the thread safety of interior mutability types.

| Type | Interior Mutability | Sync? | Thread-Safe? | Use Case |
|------|-------------------|-------|-------------|----------|
| `Cell<T>` | Copy-based | No | No | Single-threaded counters, flags |
| `RefCell<T>` | Runtime borrow checking | No | No | Single-threaded complex data |
| `Mutex<T>` | Lock-based exclusive | Yes | Yes | Multi-threaded mutable state |
| `RwLock<T>` | Lock-based read/write | Yes | Yes | Read-heavy shared state |
| `AtomicT` | Hardware atomic | Yes | Yes | Lock-free counters, flags |

## Common Pitfalls

1. **Trying `Arc<RefCell<T>>` for cross-thread sharing** — `RefCell` is `!Sync`, so `Arc<RefCell<T>>` does not compile. Use `Arc<Mutex<T>>` instead. The Mutex provides the synchronization that RefCell lacks.
2. **Using `Mutex` when an atomic would suffice** — For simple counters and boolean flags, `AtomicUsize` and `AtomicBool` are faster and simpler than `Mutex<usize>` or `Mutex<bool>`. Reach for atomics when the protected value is a primitive.

## Best Practices

1. **Match the primitive to the context** — Use `Cell`/`RefCell` in single-threaded code, `Mutex`/`RwLock` in multi-threaded code, and atomics for lock-free primitives. Do not pay for synchronization you do not need.
2. **Wrap `!Sync` types in `Mutex` to share them** — If you need to share a `!Sync` type across threads, wrap it in a `Mutex`. The `Mutex<T>: Sync` bound only requires `T: Send`, not `T: Sync`.

## Summary

- All interior mutability in Rust is built on `UnsafeCell<T>`, which is `!Sync`.
- `Cell` and `RefCell` are `!Sync` — designed for single-threaded use only.
- `Mutex` and `RwLock` are `Sync` because they add lock-based synchronization.
- Atomic types provide lock-free, hardware-supported interior mutability.
- Use `Arc<Mutex<T>>`, not `Arc<RefCell<T>>`, for shared mutable state across threads.

## Code Examples

**Arc<Mutex<T>> pattern: the correct way to share mutable state across threads**

```rust
use std::sync::{Arc, Mutex};
use std::thread;

// WRONG: Arc<RefCell<T>> does not compile — RefCell is !Sync
// let shared = Arc::new(std::cell::RefCell::new(0));

// CORRECT: Arc<Mutex<T>> provides thread-safe interior mutability
let shared = Arc::new(Mutex::new(0));

thread::scope(|s| {
    for _ in 0..10 {
        let shared = Arc::clone(&shared);
        s.spawn(move || {
            *shared.lock().unwrap() += 1;
        });
    }
});

println!("Final: {}", *shared.lock().unwrap()); // 10
```


## Resources

- [std::cell module](https://doc.rust-lang.org/std/cell/index.html) — Standard library reference for Cell, RefCell, and UnsafeCell
- [Rust Atomics and Locks — Chapter 1: Interior Mutability](https://marabos.nl/atomics/basics.html#interior-mutability) — Mara Bos's explanation of interior mutability and its relationship to Send/Sync

---

> 📘 *This lesson is part of the [Concurrent Rust](https://stanza.dev/courses/rust-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*