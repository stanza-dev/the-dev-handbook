---
source_course: "rust-concurrency"
source_lesson: "rust-concurrency-sync-trait"
---

# The Sync Trait

## Introduction

`Sync` is the companion to `Send`. While `Send` governs moving values between threads, `Sync` governs sharing references between threads. A type `T` is `Sync` if and only if `&T` is `Send` — meaning multiple threads can safely hold shared references to the same value simultaneously. This trait is the compile-time foundation for safe concurrent reads.

## Key Concepts

- **Sync**: A marker trait indicating that shared references (`&T`) to the type can be safely sent to other threads. If `T: Sync`, multiple threads can read from `&T` concurrently.
- **The equivalence**: `T: Sync` if and only if `&T: Send`. This is not just a mental model — it is the formal definition. If you can safely send a reference to another thread, the type is Sync.
- **!Sync types**: Types that allow mutation through shared references without synchronization: `Cell<T>`, `RefCell<T>`, `Rc<T>`, and the primitive `UnsafeCell<T>`. These use interior mutability without atomic operations or locks, making concurrent access unsafe.
- **Mutex makes things Sync**: `Mutex<T>` is `Sync` whenever `T: Send`, even if `T` itself is not `Sync`. The mutex ensures only one thread accesses `T` at a time, providing the synchronization that `T` lacks.

## Real World Context

When you place data behind an `Arc`, the compiler checks that `T: Sync` (because `Arc` lets multiple threads hold `&T` through its `Deref` implementation). This is why `Arc<RefCell<T>>` does not compile — `RefCell` is `!Sync` because its runtime borrow checking is not thread-safe. Understanding `Sync` tells you exactly which types can live inside an `Arc` and which need a `Mutex` wrapper.

## Deep Dive

The defining relationship is simple: `T` is `Sync` if a shared reference `&T` can be safely sent to another thread.

```rust
fn assert_sync<T: Sync>() {}

assert_sync::<i32>();         // immutable reads are always safe
assert_sync::<String>();      // String is Sync
assert_sync::<Vec<u8>>();     // Vec is Sync
assert_sync::<std::sync::Mutex<i32>>(); // Mutex is Sync
// assert_sync::<std::cell::Cell<i32>>(); // compile error: Cell is !Sync
// assert_sync::<std::cell::RefCell<i32>>(); // compile error: RefCell is !Sync
// assert_sync::<std::rc::Rc<i32>>(); // compile error: Rc is !Sync
```

The following table shows the `Send` and `Sync` status of the most important standard library types, and the reasoning behind each.

| Type | Send | Sync | Reasoning |
|------|------|------|----------|
| `i32`, `f64`, `bool` | Yes | Yes | Primitive types with no interior state |
| `String`, `Vec<T>` | Yes (if T: Send) | Yes (if T: Sync) | Owned heap data, no interior mutability |
| `Rc<T>` | No | No | Non-atomic reference count |
| `Arc<T>` | Yes (if T: Send + Sync) | Yes (if T: Send + Sync) | Atomic reference count |
| `Cell<T>` | Yes (if T: Send) | No | Interior mutability without synchronization |
| `RefCell<T>` | Yes (if T: Send) | No | Runtime borrow tracking is not atomic |
| `Mutex<T>` | Yes (if T: Send) | Yes (if T: Send) | Lock provides synchronization |
| `RwLock<T>` | Yes (if T: Send) | Yes (if T: Send + Sync) | Lock provides synchronization |
| `MutexGuard<'a, T>` | No | Yes (if T: Sync) | Must be dropped on the locking thread |
| `UnsafeCell<T>` | Yes (if T: Send) | No | The raw building block for interior mutability |

`Cell` and `RefCell` are `!Sync` because they allow mutation through `&self` (interior mutability) without any synchronization mechanism. If two threads held `&Cell<i32>` simultaneously, both could call `.set()` at the same time, causing a data race.

```rust
use std::cell::Cell;
use std::thread;

let cell = Cell::new(0);
// If Cell were Sync, we could do this:
// thread::scope(|s| {
//     s.spawn(|| cell.set(1));  // mutation via &Cell
//     s.spawn(|| cell.set(2));  // concurrent mutation — data race!
// });
// The compiler prevents this because Cell is !Sync.
```

`Mutex<T>` is `Sync` even when `T` is `!Sync`, because the mutex ensures only one thread can access the inner value at any time. The bound is `T: Send`, not `T: Sync` — the data must be transferable between threads (since any thread can acquire the lock), but it does not need to support concurrent access (the lock prevents that).

```rust
use std::sync::{Arc, Mutex};
use std::cell::RefCell;
use std::thread;

// RefCell is !Sync, but Mutex<RefCell<T>> is Sync.
let shared = Arc::new(Mutex::new(RefCell::new(vec![1, 2, 3])));

thread::scope(|s| {
    for i in 0..3 {
        let shared = Arc::clone(&shared);
        s.spawn(move || {
            let guard = shared.lock().unwrap();
            let mut borrow = guard.borrow_mut();
            borrow.push(i * 10);
            println!("thread {i}: {:?}", *borrow);
        });
    }
});
```

The equivalence `T: Sync` means `&T: Send` is visible in scoped threads. When you borrow local data in a scoped thread, the compiler checks that the reference type is `Send`, which is equivalent to the data type being `Sync`.

```rust
use std::thread;

let data = vec![1, 2, 3, 4, 5];

thread::scope(|s| {
    // &Vec<i32> is Send because Vec<i32> is Sync.
    // Multiple threads can safely hold &Vec<i32>.
    s.spawn(|| println!("thread 1: {:?}", &data));
    s.spawn(|| println!("thread 2: {:?}", &data));
});
```

`Arc<T>` requires `T: Send + Sync` because `Arc` allows multiple threads to hold a shared reference to `T`. If `T` were `!Sync`, concurrent `&T` access would be unsafe.

```rust
use std::sync::Arc;

// Works: i32 is Send + Sync
let a: Arc<i32> = Arc::new(42);

// Works: Mutex<T> is Send + Sync when T: Send
use std::sync::Mutex;
let b: Arc<Mutex<Vec<String>>> = Arc::new(Mutex::new(vec![]));

// Does NOT compile: RefCell is !Sync
// let c: Arc<RefCell<i32>> = Arc::new(RefCell::new(0));
// Error: `RefCell<i32>` cannot be shared between threads safely
```

## Common Pitfalls

1. **Assuming Arc makes anything thread-safe** — `Arc<T>` requires `T: Send + Sync`. If `T` is `!Sync` (like `RefCell`), you must wrap it in a `Mutex` first: `Arc<Mutex<RefCell<T>>>`, or better yet, just use `Arc<Mutex<T>>` directly since `Mutex` already provides interior mutability.
2. **Confusing Sync with immutability** — `Sync` does not mean immutable. `Mutex<T>` is `Sync` and provides mutable access. `Sync` means that whatever concurrent access the type permits is properly synchronized.

## Best Practices

1. **Use the relationship table as a quick reference** — When the compiler rejects a type in a concurrent context, check whether it is `Send`, `Sync`, or both. The table above covers the most common cases.
2. **Wrap !Sync types in Mutex for thread sharing** — If you need to share a `!Sync` type (like `Cell` or `RefCell`) across threads, wrap it in `Mutex` or `RwLock` to provide the missing synchronization.

## Summary

- `T: Sync` means `&T: Send` — shared references can be safely sent to other threads.
- `Cell`, `RefCell`, `Rc`, and `UnsafeCell` are `!Sync` because they allow unsynchronized interior mutation.
- `Mutex<T>` is `Sync` when `T: Send`, even if `T` is `!Sync` — the lock provides synchronization.
- `Arc<T>` requires `T: Send + Sync`; use `Arc<Mutex<T>>` for types that are `!Sync`.
- The `Send`/`Sync` system is Rust's compile-time guarantee against data races.

## Code Examples

**Sync trait verification and Arc<Mutex<T>> for sharing mutable data across threads**

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn assert_sync<T: Sync>() {}
fn assert_send<T: Send>() {}

fn main() {
    // T: Sync means &T: Send
    assert_sync::<i32>();
    assert_sync::<String>();
    assert_sync::<Mutex<i32>>();

    // Cell is !Sync — cannot be shared via &Cell across threads
    // assert_sync::<std::cell::Cell<i32>>(); // compile error

    // Arc requires T: Send + Sync
    let shared = Arc::new(Mutex::new(vec![1, 2, 3]));

    thread::scope(|s| {
        for i in 0..3 {
            let shared = Arc::clone(&shared);
            s.spawn(move || {
                let mut guard = shared.lock().unwrap();
                guard.push(i * 10);
                println!("thread {i}: {:?}", *guard);
            });
        }
    });

    println!("final: {:?}", *shared.lock().unwrap());
}
```


## Resources

- [std::marker::Sync](https://doc.rust-lang.org/std/marker/trait.Sync.html) — Standard library reference for the Sync trait
- [Send and Sync — The Rustonomicon](https://doc.rust-lang.org/nomicon/send-and-sync.html) — Unsafe Rust guide on the formal definitions of Send and Sync
- [Rust Atomics and Locks — Chapter 1](https://marabos.nl/atomics/basics.html) — Mara Bos's explanation of Send, Sync, and interior mutability

---

> 📘 *This lesson is part of the [Concurrent Rust](https://stanza.dev/courses/rust-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*