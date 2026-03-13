---
source_course: "rust-concurrency"
source_lesson: "rust-concurrency-std-spawn"
---

# Spawning OS Threads

## Introduction

Rust maps every `std::thread::spawn` call directly to an operating-system thread. This means you get real preemptive scheduling and true parallelism on multi-core hardware, but each thread carries the cost of its own stack (typically 2-8 MB). Understanding how to spawn, configure, and join threads is the foundation of all concurrent Rust programming.

## Key Concepts

- **OS Thread**: A kernel-managed thread with its own stack and register state. Rust threads are 1:1 with OS threads — there is no green-thread runtime in `std`.
- **JoinHandle<T>**: The handle returned by `thread::spawn`. It lets you wait for the thread to finish and retrieve its return value.
- **Move Closure**: A closure prefixed with `move` that takes ownership of captured variables, which is usually required when passing data into a spawned thread.
- **Thread Builder**: `thread::Builder` provides a fallible API to configure thread name and stack size before spawning.

## Real World Context

Server applications spawn threads for CPU-heavy background work — image processing, log compression, or batch computations — while the main thread continues handling requests. Game engines commonly dedicate threads to physics, audio, and rendering. Knowing how to spawn threads and retrieve their results is the entry point for all of these patterns.

## Deep Dive

The simplest way to create a thread is `thread::spawn`, which takes a closure and returns a `JoinHandle`.

```rust
use std::thread;
use std::time::Duration;

let handle = thread::spawn(|| {
    for i in 1..10 {
        println!("spawned thread: {i}");
        thread::sleep(Duration::from_millis(1));
    }
    42 // return value
});

// Main thread continues in parallel
for i in 1..5 {
    println!("main thread: {i}");
    thread::sleep(Duration::from_millis(1));
}

let result = handle.join().unwrap(); // blocks until the thread finishes
println!("thread returned: {result}");
```

The closure passed to `spawn` must be `'static + Send`, which means it cannot borrow local stack variables. To pass owned data into the thread, use a `move` closure.

```rust
let data = vec![1, 2, 3];

let handle = thread::spawn(move || {
    // `data` has been moved into this closure
    let sum: i32 = data.iter().sum();
    println!("sum = {sum}");
    sum
});
// `data` is no longer accessible here — ownership was transferred

let sum = handle.join().unwrap();
```

For more control over thread creation, use `thread::Builder`. Unlike `spawn`, it returns a `Result` so you can handle OS-level failures (such as resource exhaustion) without panicking.

```rust
let handle = thread::Builder::new()
    .name("worker-1".to_string())
    .stack_size(4 * 1024 * 1024) // 4 MB stack
    .spawn(|| {
        println!("thread name: {:?}", thread::current().name());
    })
    .expect("OS failed to create thread");

handle.join().unwrap();
```

You can query runtime information about threads at any time.

```rust
let current = thread::current();
println!("id: {:?}", current.id());
println!("name: {:?}", current.name());

// Detect available hardware parallelism
let cpus = thread::available_parallelism()
    .map(|n| n.get())
    .unwrap_or(1);
println!("available cores: {cpus}");
```

A common pattern is to fan out work across many threads and collect the results.

```rust
let handles: Vec<_> = (0..10)
    .map(|i| {
        thread::spawn(move || {
            let result = i * i;
            println!("thread {i}: {result}");
            result
        })
    })
    .collect();

let results: Vec<_> = handles
    .into_iter()
    .map(|h| h.join().unwrap())
    .collect();

println!("results: {results:?}");
```

## Common Pitfalls

1. **Forgetting to join** — If you drop a `JoinHandle` without calling `join()`, the thread is *detached*. It will keep running, but you lose the ability to wait for it or retrieve its result. This can cause the program to exit before the thread finishes.
2. **Trying to borrow local variables** — `thread::spawn` requires a `'static` closure. If you need to borrow stack data, use scoped threads (`thread::scope`) instead of `spawn`.

## Best Practices

1. **Use `Builder::spawn` in libraries** — `thread::spawn` panics if the OS cannot create the thread. Libraries should use `Builder::spawn` and propagate the `io::Error` to the caller.
2. **Limit thread count** — Creating thousands of OS threads wastes memory and hurts performance. Use `available_parallelism()` to size your thread pool appropriately.

## Summary

- `thread::spawn` creates a 1:1 OS thread and returns a `JoinHandle<T>`.
- Move closures transfer ownership of captured variables into the thread.
- `thread::Builder` gives you control over thread name, stack size, and error handling.
- Always `join()` handles you care about to avoid detached threads and lost results.

## Code Examples

**Fan-out pattern: spawn multiple threads and collect their results into a Vec**

```rust
use std::thread;

// Spawn multiple threads and collect results
let handles: Vec<_> = (0..10)
    .map(|i| {
        thread::spawn(move || {
            let result = i * i;
            println!("Thread {i}: {result}");
            result
        })
    })
    .collect();

let results: Vec<_> = handles
    .into_iter()
    .map(|h| h.join().unwrap())
    .collect();

println!("Results: {results:?}");
```


## Resources

- [Using Threads to Run Code Simultaneously](https://doc.rust-lang.org/book/ch16-01-threads.html) — Official Rust Book chapter on OS threads
- [std::thread module](https://doc.rust-lang.org/std/thread/index.html) — Standard library reference for thread APIs

---

> 📘 *This lesson is part of the [Concurrent Rust](https://stanza.dev/courses/rust-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*