---
source_course: "rust-concurrency"
source_lesson: "rust-concurrency-thread-safety-patterns"
---

# Thread Safety Patterns

## Introduction

Knowing what `Send` and `Sync` mean is only the first step. In practice, you need patterns for controlling these traits in your own types, choosing the right concurrency primitive, and auditing thread safety. This lesson covers the essential patterns that experienced Rust developers use daily: `PhantomData` for trait control, newtype wrappers, negative trait implementations, and the decision tree for choosing between `Arc<Mutex<T>>` and `Arc<RwLock<T>>`.

## Key Concepts

- **PhantomData**: A zero-sized type that tells the compiler your struct logically "contains" a type `T` without actually storing it. By embedding `PhantomData<*const T>` (a raw pointer, which is `!Send + !Sync`), you can opt a type out of `Send` and `Sync` auto-implementation.
- **Newtype pattern**: Wrapping a type in a single-field struct to change its `Send`/`Sync` behavior. Useful when you need to add `Send` or `Sync` to a foreign type or restrict it.
- **Negative impls**: `impl !Send for Type {}` and `impl !Sync for Type {}` explicitly opt a type out of `Send` or `Sync`. These require nightly Rust (the `negative_impls` feature), so stable code uses `PhantomData` as the alternative.
- **Interior mutability and Sync**: Any type that allows mutation through a shared reference (`&self`) must provide its own synchronization to be `Sync`. `UnsafeCell<T>` is the fundamental building block — it is `!Sync`, and types like `Cell`, `RefCell`, `Mutex`, and `RwLock` all build on top of it.
- **Scoped threads and 'static**: `thread::spawn` requires `'static` data, but `thread::scope` relaxes this requirement by guaranteeing all threads join before the scope exits. This eliminates the need for `Arc` in many fork-join patterns.

## Real World Context

Library authors frequently need to control `Send`/`Sync` for wrapper types around FFI handles or hardware resources. A GPU buffer handle, for example, must not be sent to another thread if the graphics API is single-threaded. Using `PhantomData<*const ()>` ensures the compiler enforces this. Application developers face the `Arc<Mutex<T>>` vs `Arc<RwLock<T>>` decision daily — choosing wrong can halve throughput or introduce unnecessary complexity.

## Deep Dive

### PhantomData for Send/Sync Control

`PhantomData` is a zero-sized marker that affects trait auto-implementation without adding any runtime cost.

```rust
use std::marker::PhantomData;

// This struct wraps a raw file descriptor.
// It should NOT be Send or Sync because the FD is tied to the current thread's I/O context.
struct ThreadLocalHandle {
    fd: i32,
    _not_send_sync: PhantomData<*const ()>,
    // *const () is !Send and !Sync, so ThreadLocalHandle becomes !Send and !Sync.
}

impl ThreadLocalHandle {
    fn new(fd: i32) -> Self {
        Self {
            fd,
            _not_send_sync: PhantomData,
        }
    }
}

// Verify at compile time:
fn assert_not_send<T>() where T: Send {} // This would fail for ThreadLocalHandle
// assert_not_send::<ThreadLocalHandle>(); // compile error: not Send
```

By including `PhantomData<*const ()>`, which is `!Send` and `!Sync`, the compiler treats the entire struct as if it contains a raw pointer, preventing it from being moved between threads or shared via references.

You can also use `PhantomData` to make a type `!Send` but still `Sync`, or vice versa, by choosing the right phantom type.

```rust
use std::marker::PhantomData;

// !Send but Sync: use PhantomData<std::sync::MutexGuard<'static, ()>>
// MutexGuard is !Send but Sync.
struct PinnedToThread {
    data: i32,
    _no_send: PhantomData<std::sync::MutexGuard<'static, ()>>,
}
// PinnedToThread is !Send (can't move between threads)
// but IS Sync (can share &PinnedToThread between threads)
```

This technique gives you precise control over which traits are opted out, since different phantom types carry different trait implementations.

### Newtype Pattern for Thread Safety

The newtype pattern wraps an existing type to change its thread safety characteristics.

```rust
use std::sync::Arc;

// Suppose you have a C library handle that you've audited and know is thread-safe,
// but Rust marks it as !Send because it contains a raw pointer.
struct FfiHandle {
    ptr: *mut std::ffi::c_void,
}

// Newtype wrapper that asserts thread safety
struct ThreadSafeHandle(FfiHandle);

// SAFETY: The underlying C library is thread-safe.
// All functions that operate on this handle use internal locking.
unsafe impl Send for ThreadSafeHandle {}
unsafe impl Sync for ThreadSafeHandle {}

// Now ThreadSafeHandle can be used with Arc and sent between threads.
let handle = Arc::new(ThreadSafeHandle(FfiHandle { ptr: std::ptr::null_mut() }));
```

The newtype wrapper isolates the unsafe Send/Sync implementations to a single type with a clear safety comment, keeping the rest of your codebase safe.

### Auditing a Type for Thread Safety

When deciding whether to implement `Send` or `Sync` for a type, ask these questions in order.

```rust
// Thread safety audit checklist:
//
// 1. Does the type contain raw pointers or references to non-Send/Sync types?
//    -> If yes, can you PROVE the pointed-to data is safe to access from other threads?
//
// 2. Does the type have interior mutability (mutation through &self)?
//    -> If yes, is that mutation synchronized (atomics, locks)?
//    -> If not synchronized, the type must be !Sync.
//
// 3. Does the type interact with thread-local state, signal handlers, or OS handles?
//    -> If yes, it should probably be !Send.
//
// 4. Does the type have a destructor that must run on a specific thread?
//    -> If yes, it must be !Send.

// Example audit: a connection pool
struct ConnectionPool {
    connections: std::sync::Mutex<Vec<Connection>>,
    max_size: usize,
}
// Audit:
// - Contains Mutex<Vec<Connection>>: Mutex is Send+Sync if Connection is Send. ✓
// - max_size is usize: Send+Sync. ✓
// - No raw pointers, no thread-local state, no thread-specific destructor.
// Result: ConnectionPool is automatically Send+Sync (if Connection: Send). ✓
```

Walking through this checklist for every type that wraps raw pointers or FFI handles ensures you make an informed decision about thread safety rather than blindly adding `unsafe impl Send`.

### Negative Impls (Nightly)

On nightly Rust, you can explicitly opt out of `Send` or `Sync` using negative trait implementations. This is clearer than the `PhantomData` workaround but is not yet stabilized.

```rust
// Requires: #![feature(negative_impls)]

struct GpuBuffer {
    handle: u64,
}

// Explicitly mark as !Send — the GPU context is thread-local.
impl !Send for GpuBuffer {}
// Explicitly mark as !Sync — no concurrent access allowed.
impl !Sync for GpuBuffer {}
```

Negative impls are more self-documenting than `PhantomData` because they state the intent explicitly. However, since they require nightly, most libraries use the `PhantomData` approach instead.

On stable Rust, use `PhantomData<*const ()>` to achieve the same effect, as shown earlier.

### Interior Mutability and Sync

`UnsafeCell<T>` is the only way to obtain a `&mut T` from a `&T` in Rust. Every interior mutability type builds on it. Because `UnsafeCell` is `!Sync`, any type that wraps it must provide its own synchronization to be `Sync`.

```rust
use std::cell::UnsafeCell;

// UnsafeCell is !Sync — the compiler forces you to add synchronization.
struct MyMutex<T> {
    locked: std::sync::atomic::AtomicBool,
    data: UnsafeCell<T>,
}

// We must manually implement Sync, providing our own synchronization.
// SAFETY: Access to `data` is guarded by the `locked` atomic flag.
unsafe impl<T: Send> Sync for MyMutex<T> {}
```

This shows the fundamental pattern: `UnsafeCell` opts out of `Sync`, and then `MyMutex` opts back in by providing its own atomic synchronization via the `locked` flag. The `unsafe impl` is the programmer's promise that the synchronization is correct.

### Scoped Threads: Relaxing 'static

`thread::spawn` requires `'static` data, which typically means using `Arc` for shared ownership. `thread::scope` relaxes this requirement by guaranteeing all threads join before the scope exits.

```rust
use std::thread;

let mut data = vec![0; 1000];
let chunk_size = data.len() / 4;

// No Arc needed — scoped threads can borrow local data.
thread::scope(|s| {
    for (i, chunk) in data.chunks_mut(chunk_size).enumerate() {
        s.spawn(move || {
            for x in chunk.iter_mut() {
                *x = i as i32;
            }
        });
    }
});
// All threads have joined. `data` is fully populated.
println!("{:?}", &data[..8]);
```

This eliminates the `Arc` boilerplate entirely. Each thread gets a `&mut` slice of the data, and the scope guarantees the borrows are valid for the duration of the parallel work.

Scoped threads still require that borrowed types are `Sync` (for shared borrows) or `Send` (for mutable borrows). The difference is that `'static` is no longer required.

### Arc<Mutex<T>> vs Arc<RwLock<T>> Decision Tree

Choosing between `Mutex` and `RwLock` depends on your access pattern.

```rust
// Decision tree:
//
// 1. Are reads much more frequent than writes (>80% reads)?
//    YES -> Use Arc<RwLock<T>>
//    NO  -> Use Arc<Mutex<T>> (simpler, lower overhead)
//
// 2. Is the critical section very short (< 1 microsecond)?
//    YES -> Consider parking_lot::Mutex (no poisoning, smaller, faster)
//    NO  -> std::sync::Mutex is fine
//
// 3. Do you need to downgrade write locks to read locks?
//    YES -> Use RwLock (RwLockWriteGuard::downgrade, Rust 1.92+)
//    NO  -> Either works
//
// 4. Do you need try_lock with a timeout?
//    YES -> parking_lot provides try_lock_for() and try_lock_until()
//    NO  -> std is fine
//
// 5. Do you want to avoid poisoning?
//    YES -> Use parking_lot (no poisoning by design)
//    NO  -> std is fine

use std::sync::{Arc, Mutex, RwLock};

// Write-heavy: use Mutex
let counter = Arc::new(Mutex::new(0u64));

// Read-heavy: use RwLock
let config = Arc::new(RwLock::new(AppConfig::default()));
```

This decision tree provides a practical framework: start with the simplest option (Mutex), and only add complexity (RwLock, parking_lot) when profiling shows a concrete bottleneck.

### When to Use parking_lot Alternatives

The `parking_lot` crate provides drop-in replacements for `Mutex`, `RwLock`, `Condvar`, and `Once` with several advantages.

```rust
use parking_lot::{Mutex, RwLock};

// parking_lot advantages over std:
// 1. No poisoning — lock() returns the guard directly, not a Result.
// 2. Smaller — parking_lot::Mutex is 1 byte, std::sync::Mutex is 40+ bytes.
// 3. Faster — optimized lock algorithms, especially under contention.
// 4. Fair locking option — RwLock can be configured for writer-priority or fair.
// 5. Timed locking — try_lock_for(Duration) and try_lock_until(Instant).

let m = Mutex::new(42);
let guard = m.lock(); // returns MutexGuard directly, no unwrap() needed
println!("{}", *guard);

// Timed lock attempt
use std::time::Duration;
if let Some(guard) = m.try_lock_for(Duration::from_millis(100)) {
    println!("got lock: {}", *guard);
} else {
    println!("lock timeout");
}
```

The `parking_lot` API is notably simpler: `lock()` returns the guard directly instead of a `Result`, since `parking_lot` does not use poisoning. This eliminates the `.unwrap()` calls that litter `std::sync::Mutex` usage.

Use `parking_lot` when you want simpler APIs (no poisoning), better performance under contention, or timed lock acquisition. Use `std::sync` when you want zero external dependencies or need poisoning for panic safety.

## Common Pitfalls

1. **Using PhantomData<T> instead of PhantomData<*const T>** — `PhantomData<T>` makes your type act as if it *owns* a `T`, which affects drop checking but does not remove `Send`/`Sync`. Use `PhantomData<*const T>` or `PhantomData<*const ()>` to opt out of `Send` and `Sync`.
2. **Choosing RwLock for write-heavy workloads** — `RwLock` has higher per-operation overhead than `Mutex` because it tracks reader counts. If writes are frequent, `Mutex` is simpler and often faster.

## Best Practices

1. **Default to Arc<Mutex<T>> and optimize later** — `Mutex` is simpler, has lower overhead, and works for any access pattern. Switch to `RwLock` only when profiling shows that read contention is a bottleneck.
2. **Use PhantomData on stable, negative impls on nightly** — For library code that targets stable Rust, `PhantomData<*const ()>` is the standard way to opt out of `Send`/`Sync`. Document the reason in a comment.

## Summary

- `PhantomData<*const ()>` removes `Send` and `Sync` from a type at zero runtime cost.
- The newtype pattern lets you add or remove thread safety traits for foreign types.
- Negative impls (`impl !Send`) exist on nightly; use `PhantomData` on stable.
- `UnsafeCell` is the root of all interior mutability and is `!Sync` by definition.
- Scoped threads relax `'static` requirements, often eliminating the need for `Arc`.
- Default to `Arc<Mutex<T>>`; switch to `Arc<RwLock<T>>` only for read-heavy workloads.
- `parking_lot` offers smaller, faster, non-poisoning lock primitives with timed locking.

## Code Examples

**PhantomData for trait control, Arc<Mutex<T>> vs Arc<RwLock<T>> usage patterns**

```rust
use std::marker::PhantomData;
use std::sync::{Arc, Mutex, RwLock};
use std::thread;

// Pattern 1: PhantomData to remove Send/Sync
struct GpuHandle {
    id: u64,
    _not_send: PhantomData<*const ()>, // makes GpuHandle !Send + !Sync
}

// Pattern 2: Arc<Mutex<T>> for write-heavy shared state
fn mutex_example() {
    let counter = Arc::new(Mutex::new(0u64));
    thread::scope(|s| {
        for _ in 0..8 {
            let counter = Arc::clone(&counter);
            s.spawn(move || {
                for _ in 0..1000 {
                    *counter.lock().unwrap() += 1;
                }
            });
        }
    });
    println!("counter: {}", *counter.lock().unwrap()); // 8000
}

// Pattern 3: Arc<RwLock<T>> for read-heavy shared state
fn rwlock_example() {
    let config = Arc::new(RwLock::new(vec!["default".to_string()]));
    thread::scope(|s| {
        // Many readers
        for i in 0..8 {
            let config = Arc::clone(&config);
            s.spawn(move || {
                let data = config.read().unwrap();
                println!("reader {i}: {:?}", *data);
            });
        }
        // One writer
        let config = Arc::clone(&config);
        s.spawn(move || {
            config.write().unwrap().push("updated".to_string());
        });
    });
}

fn main() {
    mutex_example();
    rwlock_example();
}
```


## Resources

- [std::marker::PhantomData](https://doc.rust-lang.org/std/marker/struct.PhantomData.html) — Standard library reference for PhantomData and its effects on auto-traits
- [parking_lot crate](https://docs.rs/parking_lot/) — High-performance synchronization primitives — drop-in replacements for std locks
- [Rust Atomics and Locks — Chapter 1: Basics](https://marabos.nl/atomics/basics.html) — Interior mutability, UnsafeCell, and the foundations of thread safety in Rust
- [std::cell::UnsafeCell](https://doc.rust-lang.org/std/cell/struct.UnsafeCell.html) — The fundamental building block for interior mutability

---

> 📘 *This lesson is part of the [Concurrent Rust](https://stanza.dev/courses/rust-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*