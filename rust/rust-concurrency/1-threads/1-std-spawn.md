---
source_course: "rust-concurrency"
source_lesson: "rust-concurrency-std-spawn"
---

# std::thread

Rust threads map 1:1 to OS threads - they're real, heavyweight threads.

```rust
use std::thread;
use std::time::Duration;

let handle = thread::spawn(|| {
    for i in 1..10 {
        println!("Hi from spawned thread: {i}");
        thread::sleep(Duration::from_millis(1));
    }
});

// Main thread work
for i in 1..5 {
    println!("Hi from main thread: {i}");
    thread::sleep(Duration::from_millis(1));
}

handle.join().unwrap(); // Wait for spawned thread
```

## Move Closures

To use data from the main thread, use `move`:

```rust
let v = vec![1, 2, 3];

let handle = thread::spawn(move || {
    println!("{v:?}");
});
// v is no longer accessible here - it was moved!

handle.join().unwrap();
```

## Thread Builder

```rust
let handle = thread::Builder::new()
    .name("worker-1".to_string())
    .stack_size(4 * 1024 * 1024)  // 4 MB stack
    .spawn(|| {
        println!("Thread name: {:?}", thread::current().name());
    })
    .expect("Failed to spawn thread");
```

## Getting Thread Info

```rust
let current = thread::current();
println!("Thread ID: {:?}", current.id());
println!("Thread name: {:?}", current.name());

// Available parallelism (number of CPUs)
let cpus = thread::available_parallelism().unwrap().get();
println!("CPUs: {cpus}");
```

See [Using Threads](https://doc.rust-lang.org/book/ch16-01-threads.html).

## Code Examples

**Multiple threads with results**

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

- [Using Threads](https://doc.rust-lang.org/book/ch16-01-threads.html) — Official guide to threads

---

> 📘 *This lesson is part of the [Concurrent Rust](https://stanza.dev/courses/rust-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*