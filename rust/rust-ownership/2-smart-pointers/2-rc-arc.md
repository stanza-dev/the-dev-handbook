---
source_course: "rust-ownership"
source_lesson: "rust-ownership-rc-arc"
---

# Rc<T> & Arc<T>: Shared Ownership

## Introduction
Rust's ownership model normally enforces single ownership, but some data structures require multiple owners — graph nodes, shared caches, or configuration shared across threads. Rc and Arc provide reference-counted shared ownership for single-threaded and multi-threaded contexts respectively.

## Key Concepts
- **Rc<T>**: Reference Counted pointer for single-threaded shared ownership. Cloning increments the count; dropping decrements it. Data is freed when the count reaches zero.
- **Arc<T>**: Atomically Reference Counted pointer, the thread-safe version of Rc that uses atomic operations for the reference count.
- **Strong count**: The number of owning references. Data is deallocated when this reaches zero.

## Real World Context
Graph data structures where nodes have multiple parents, observer patterns where multiple listeners hold references to a shared state, and shared configuration passed to multiple subsystems all require shared ownership. In multi-threaded servers, Arc is used to share read-only configuration across worker threads.

## Deep Dive
Rc allows multiple variables to own the same heap-allocated data:

```rust
use std::rc::Rc;

let a = Rc::new(String::from("hello"));
let b = Rc::clone(&a);  // Increments count, does NOT clone the String
let c = Rc::clone(&a);

println!("Count: {}", Rc::strong_count(&a));  // 3
// Data is freed when a, b, and c are all dropped
```

Rc is useful for shared graph nodes where a node has multiple parents:

```rust
use std::rc::Rc;

struct Node {
    value: i32,
    children: Vec<Rc<Node>>,
}

let shared = Rc::new(Node { value: 1, children: vec![] });
let parent1 = Node { value: 2, children: vec![Rc::clone(&shared)] };
let parent2 = Node { value: 3, children: vec![Rc::clone(&shared)] };
// shared has two parents — both own a reference to it
```

For multi-threaded code, use Arc which uses atomic operations for thread safety:

```rust
use std::sync::Arc;
use std::thread;

let data = Arc::new(vec![1, 2, 3]);

let handles: Vec<_> = (0..3).map(|_| {
    let data = Arc::clone(&data);
    thread::spawn(move || {
        println!("{:?}", data);
    })
}).collect();

for h in handles { h.join().unwrap(); }
```

Neither Rc nor Arc allows mutation. To mutate shared data, combine them with interior mutability types: `Rc<RefCell<T>>` or `Arc<Mutex<T>>`.

## Common Pitfalls
1. **Using Rc across threads** — Rc is `!Send` and `!Sync`. Attempting to send an Rc to another thread is a compile error. Use Arc instead.
2. **Creating reference cycles with Rc** — Two Rc values pointing to each other will never be freed. Use `Weak` references to break cycles.

## Best Practices
1. **Default to single ownership** — Only reach for Rc/Arc when you genuinely need multiple owners. Most Rust code works with plain ownership or borrowing.
2. **Use Arc only when crossing thread boundaries** — Arc's atomic operations have overhead compared to Rc. Use Rc when all access is single-threaded.

## Summary
- Rc<T> enables shared ownership in single-threaded code via reference counting.
- Arc<T> is the thread-safe version using atomic operations.
- Neither allows mutation; combine with RefCell or Mutex for interior mutability.
- Cloning only increments the reference count, not the underlying data.

## Code Examples

**Sharing configuration across threads with Arc — each thread gets a clone of the Arc, incrementing the reference count**

```rust
use std::sync::Arc;
use std::thread;

// Sharing read-only config across threads
struct Config {
    max_connections: usize,
    timeout_ms: u64,
}

let config = Arc::new(Config { max_connections: 100, timeout_ms: 5000 });

let handles: Vec<_> = (0..4).map(|i| {
    let config = Arc::clone(&config);
    thread::spawn(move || {
        println!("Worker {i}: max={}", config.max_connections);
    })
}).collect();

for h in handles { h.join().unwrap(); }
```


## Resources

- [Rc<T> Documentation](https://doc.rust-lang.org/std/rc/struct.Rc.html) — Official API reference for Rc
- [Arc<T> Documentation](https://doc.rust-lang.org/std/sync/struct.Arc.html) — Official API reference for Arc

---

> 📘 *This lesson is part of the [Rust Ownership & Memory Model](https://stanza.dev/courses/rust-ownership) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*