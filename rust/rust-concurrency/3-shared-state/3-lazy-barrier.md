---
source_course: "rust-concurrency"
source_lesson: "rust-concurrency-once-lazy-barrier"
---

# Once, LazyLock & Barrier

## Introduction

Some data only needs to be initialized once and then shared immutably across threads. Rust provides a family of one-time initialization primitives — `Once`, `OnceLock`, `LazyCell`, and `LazyLock` — each suited to a different use case. `Barrier` serves a different purpose: synchronizing a group of threads so they all start work at the same moment.

## Key Concepts

- **Once**: A synchronization primitive that runs a closure exactly once, even when called from multiple threads simultaneously. Subsequent calls to `call_once` are no-ops.
- **Once::wait()**: Stabilized in Rust 1.86, this method blocks until some thread has completed the `call_once` initialization, without running the closure itself. Useful for observer threads that need the initialization to be done but should not trigger it.
- **OnceLock<T>**: A thread-safe cell that can be written to exactly once. Combines `Once` with storage for a value of type `T`. Common for lazy global initialization.
- **OnceLock::wait()**: Also stabilized in Rust 1.86, blocks until the `OnceLock` has been initialized and returns a reference to the value. Unlike `get_or_init()`, it does not accept a closure — it only observes.
- **LazyCell<T>**: Stabilized in Rust 1.80, `LazyCell` (in `std::cell`) provides lazy initialization for single-threaded use. It stores a closure and evaluates it on first access. It is `!Sync` and cannot be shared between threads.
- **LazyLock<T>**: Stabilized in Rust 1.80, `LazyLock` (in `std::sync`) is the thread-safe counterpart of `LazyCell`. It evaluates its closure exactly once on first access, even under concurrent access from multiple threads.
- **Barrier**: Blocks a group of `n` threads until all of them have called `wait()`, then releases them all simultaneously. Useful for phased computations where all threads must finish one phase before any can start the next.

## Real World Context

Global logger initialization, database connection pool setup, and regex compilation are all classic use cases for `OnceLock` and `LazyLock` — they should happen once, lazily, and be safe under concurrent access. `Barrier` is common in scientific and simulation code where you need all worker threads to synchronize between computation steps (e.g., each iteration of a physics simulation must complete before the next begins).

## Deep Dive

### Once and call_once

`Once` guarantees that the closure passed to `call_once` executes exactly once, no matter how many threads call it.

```rust
use std::sync::Once;

static INIT: Once = Once::new();

fn initialize() {
    INIT.call_once(|| {
        println!("this runs exactly once");
        // expensive setup: open files, connect to database, etc.
    });
}

// Call from many threads — only the first one runs the closure
initialize(); // prints
initialize(); // no-op
initialize(); // no-op
```

The key takeaway is that `call_once` is idempotent: no matter how many times you call it, the closure runs exactly once and all subsequent calls are fast no-ops.

### Once::wait() (Rust 1.86+)

The `wait()` method blocks the calling thread until some other thread has completed the initialization via `call_once`. It does not run the closure itself — it only waits for it to finish.

```rust
use std::sync::Once;
use std::thread;

static INIT: Once = Once::new();
static mut CONFIG: Option<String> = None;

let initializer = thread::spawn(|| {
    INIT.call_once(|| {
        // Safety: we are the only thread writing, inside call_once
        unsafe {
            CONFIG = Some("production".to_string());
        }
    });
});

// Observer thread: wait for init to complete, don't run the closure
let observer = thread::spawn(|| {
    INIT.wait(); // blocks until call_once finishes
    // Safety: initialization is guaranteed complete
    unsafe {
        println!("config: {:?}", CONFIG.as_ref().unwrap());
    }
});

initializer.join().unwrap();
observer.join().unwrap();
```

Here, `wait()` lets the observer thread block until `call_once` has completed without being responsible for running the initialization itself. This cleanly separates the "who initializes" concern from the "who needs the data" concern.

### OnceLock<T>

`OnceLock` combines `Once` with typed storage. It is the modern, safe replacement for the `Once` + `static mut` pattern shown above.

```rust
use std::sync::OnceLock;

static CONFIG: OnceLock<String> = OnceLock::new();

fn get_config() -> &'static str {
    CONFIG.get_or_init(|| {
        println!("initializing config...");
        "production".to_string()
    })
}

println!("{}", get_config()); // initializes and returns "production"
println!("{}", get_config()); // returns cached "production"
```

Notice how `OnceLock` combines the `Once` synchronization with typed storage, eliminating the need for `unsafe` and `static mut`. The `get_or_init` method handles both the first-time initialization and subsequent cache hits.

`OnceLock::wait()` (Rust 1.86+) blocks until the value is initialized and returns a reference, without triggering initialization itself.

```rust
use std::sync::OnceLock;
use std::thread;

static DB_POOL: OnceLock<String> = OnceLock::new();

let init_thread = thread::spawn(|| {
    DB_POOL.get_or_init(|| {
        thread::sleep(std::time::Duration::from_millis(100));
        "postgres://localhost/mydb".to_string()
    });
});

// This thread waits for initialization but does not trigger it
let observer = thread::spawn(|| {
    let pool = DB_POOL.wait(); // blocks until initialized
    println!("observer sees: {pool}");
});

init_thread.join().unwrap();
observer.join().unwrap();
```

This pattern is particularly useful in server startup: one thread initializes the database pool while other threads wait for it to become available before handling requests.

### LazyCell<T> (Single-Threaded)

`LazyCell` (in `std::cell`) evaluates its closure on first access. It is `!Sync`, meaning it cannot be shared between threads — it is designed for single-threaded lazy initialization.

```rust
use std::cell::LazyCell;

let lazy = LazyCell::new(|| {
    println!("computing...");
    92
});

println!("before access"); // "computing..." has NOT been printed yet
println!("value: {}", *lazy); // prints "computing..." then "value: 92"
println!("value: {}", *lazy); // prints "value: 92" — closure not called again
```

The closure is evaluated lazily on first dereference and cached for all subsequent accesses. Since `LazyCell` is `!Sync`, it cannot be placed in a `static` or shared between threads.

### LazyLock<T> (Thread-Safe)

`LazyLock` (in `std::sync`) is the thread-safe version. It is ideal for global statics that should be initialized lazily.

```rust
use std::sync::LazyLock;

static GLOBAL_REGEX: LazyLock<regex::Regex> = LazyLock::new(|| {
    regex::Regex::new(r"^\d{4}-\d{2}-\d{2}$").unwrap()
});

fn is_date(s: &str) -> bool {
    GLOBAL_REGEX.is_match(s) // initialized on first call
}
```

This is the most common use of `LazyLock`: initializing a global static value lazily and safely. The regex is compiled only once, on the first call to `is_date`, and every subsequent call reuses the cached compiled regex.

`LazyLock::force()` explicitly triggers initialization without dereferencing. This is useful when you want to ensure the value is initialized at a specific point in your program.

As of Rust 1.94, `LazyCell` and `LazyLock` both provide additional stable accessor methods: `get()` returns `Option<&T>` to check whether the value has been initialized without triggering initialization, and `force_mut()` provides mutable access to the initialized value. These methods give you fine-grained control over lazy values without needing unstable features.

```rust
use std::sync::LazyLock;

let lazy = LazyLock::new(|| String::from("hello"));

// Check without initializing
assert!(LazyLock::get(&lazy).is_none());

// Force initialization
LazyLock::force(&lazy);
assert!(LazyLock::get(&lazy).is_some());
```

The `get()` method is useful for checking initialization status without triggering it, which is handy in diagnostics or conditional logic where you want to know if a lazy value has already been computed.

### Barrier

`Barrier` blocks threads until a specified number have arrived, then releases them all at once. This is useful for phased parallel algorithms.

```rust
use std::sync::{Arc, Barrier};
use std::thread;

let num_threads = 4;
let barrier = Arc::new(Barrier::new(num_threads));

let handles: Vec<_> = (0..num_threads)
    .map(|i| {
        let barrier = Arc::clone(&barrier);
        thread::spawn(move || {
            println!("thread {i}: doing phase 1 work");
            // Simulate work
            thread::sleep(std::time::Duration::from_millis(100 * i as u64));

            println!("thread {i}: waiting at barrier");
            let result = barrier.wait();

            // All threads reach here at the same time
            println!("thread {i}: starting phase 2");

            // Exactly one thread is designated the "leader"
            if result.is_leader() {
                println!("thread {i}: I am the leader!");
            }
        })
    })
    .collect();

for h in handles {
    h.join().unwrap();
}
```

Each thread does its phase 1 work independently, then calls `barrier.wait()`. The barrier blocks each thread until all four have arrived, at which point they are all released simultaneously to begin phase 2.

The `wait()` method returns a `BarrierWaitResult`. Exactly one thread per barrier release is designated the "leader" (via `is_leader()`), which is useful for performing a single cleanup or aggregation step before the next phase.

Barriers are reusable. After all threads pass through, the barrier resets and can be used again for the next phase.

```rust
use std::sync::{Arc, Barrier};
use std::thread;

let barrier = Arc::new(Barrier::new(3));

let handles: Vec<_> = (0..3)
    .map(|i| {
        let barrier = Arc::clone(&barrier);
        thread::spawn(move || {
            for phase in 0..3 {
                println!("thread {i}: working on phase {phase}");
                barrier.wait(); // syncs after each phase
            }
        })
    })
    .collect();

for h in handles {
    h.join().unwrap();
}
```

This demonstrates that barriers are reusable: after all three threads pass through, the barrier resets automatically and can synchronize the next phase without any additional setup.

## Common Pitfalls

1. **Using `LazyCell` across threads** — `LazyCell` is `!Sync` and will cause a compile error if placed in a `static` or shared between threads. Use `LazyLock` for thread-safe lazy initialization.
2. **Barrier with wrong count** — If you create a `Barrier::new(n)` but only `n-1` threads call `wait()`, all of them will block forever. The count must exactly match the number of threads that will synchronize.

## Best Practices

1. **Prefer `LazyLock` over `OnceLock` for statics** — `LazyLock` combines storage and initialization in one declaration and evaluates automatically on first access. Use `OnceLock` when initialization needs to be triggered from a specific call site or when you need `wait()`.
2. **Use `Barrier` for phased parallel algorithms** — When all threads must complete step N before any can start step N+1, a barrier is the clearest and most efficient synchronization primitive.

## Summary

- `Once::call_once()` runs a closure exactly once; `Once::wait()` (Rust 1.86) blocks until initialization completes without running the closure.
- `OnceLock<T>` provides typed one-time initialization with `get_or_init()` and `wait()` (Rust 1.86).
- `LazyCell<T>` (Rust 1.80, `std::cell`) is for single-threaded lazy init; `LazyLock<T>` (Rust 1.80, `std::sync`) is the thread-safe variant.
- `LazyLock::force()` triggers initialization explicitly. Rust 1.94 adds stable `get()` and `force_mut()` accessors to both `LazyCell` and `LazyLock`.
- `Barrier` synchronizes N threads at a rendezvous point and designates one as the leader.

## Code Examples

**LazyLock for global config and Barrier for phased thread synchronization**

```rust
use std::sync::{Arc, Barrier, LazyLock};
use std::thread;

// LazyLock: thread-safe global initialization
static GLOBAL_CONFIG: LazyLock<Vec<String>> = LazyLock::new(|| {
    println!("Initializing config (runs once)...");
    vec!["key=value".to_string(), "debug=true".to_string()]
});

fn main() {
    let barrier = Arc::new(Barrier::new(3));

    let handles: Vec<_> = (0..3)
        .map(|i| {
            let barrier = Arc::clone(&barrier);
            thread::spawn(move || {
                // All threads can read GLOBAL_CONFIG (initialized on first access)
                println!("Thread {i}: config = {:?}", &*GLOBAL_CONFIG);

                // Barrier: wait for all threads before proceeding
                barrier.wait();
                println!("Thread {i}: all threads synced, starting phase 2");
            })
        })
        .collect();

    for h in handles {
        h.join().unwrap();
    }
}
```


## Resources

- [std::sync::Once](https://doc.rust-lang.org/std/sync/struct.Once.html) — Standard library reference for Once, including wait() (Rust 1.86)
- [std::sync::OnceLock](https://doc.rust-lang.org/std/sync/struct.OnceLock.html) — Standard library reference for OnceLock<T>
- [std::sync::LazyLock](https://doc.rust-lang.org/std/sync/struct.LazyLock.html) — Thread-safe lazy initialization, stabilized in Rust 1.80
- [std::cell::LazyCell](https://doc.rust-lang.org/std/cell/struct.LazyCell.html) — Single-threaded lazy initialization, stabilized in Rust 1.80
- [std::sync::Barrier](https://doc.rust-lang.org/std/sync/struct.Barrier.html) — Standard library reference for Barrier synchronization

---

> 📘 *This lesson is part of the [Concurrent Rust](https://stanza.dev/courses/rust-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*