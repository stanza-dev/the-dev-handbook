---
source_course: "rust-async"
source_lesson: "rust-async-spawning"
---

# Spawning Tasks

## Introduction

Tasks are the async equivalent of threads — lightweight, concurrent units of work. Tokio provides several spawn variants for different scenarios: `spawn`, `spawn_local`, `spawn_blocking`, and `JoinSet` for managing groups of tasks.

## Key Concepts

**`tokio::spawn`** — runs a future concurrently. Requires `'static + Send`:

```rust
let handle = tokio::spawn(async {
    expensive_io().await
});
let result = handle.await.unwrap();
```

**`spawn_local`** — for `!Send` futures (e.g., Rc, Cell):

```rust
let local = tokio::task::LocalSet::new();
local.run_until(async {
    tokio::task::spawn_local(async {
        let rc = std::rc::Rc::new(42);
        println!("{rc}");
    }).await.unwrap();
}).await;
```

**`spawn_blocking`** — offloads CPU-heavy or blocking sync work:

```rust
let result = tokio::task::spawn_blocking(|| {
    heavy_computation() // Runs on dedicated thread pool
}).await?;
```

## Real World Context

Web servers spawn a task per request. Background jobs use `spawn`. Image processing or crypto use `spawn_blocking`. `JoinSet` manages fan-out patterns.

## Deep Dive

**Why `'static + Send`?** Spawned tasks may outlive the spawning scope and may run on any executor thread. You must `move` ownership into the closure:

```rust
let data = String::from("hello");
tokio::spawn(async move {
    println!("{data}"); // data moved in
});
// data no longer accessible here
```

**JoinSet** manages dynamic collections of tasks:

```rust
use tokio::task::JoinSet;

let mut set = JoinSet::new();
for i in 0..10 {
    set.spawn(async move { i * 2 });
}
while let Some(result) = set.join_next().await {
    println!("Got: {:?}", result.unwrap());
}
```

**JoinHandle** returns `Result<T, JoinError>` — the error case handles panics.

## Common Pitfalls

- Borrowing local variables into spawned tasks — must `move` or `Arc`
- Ignoring JoinHandle — task panics go unnoticed
- Using `spawn_blocking` for async I/O — wastes a thread

## Best Practices

- Always handle or log JoinHandle errors
- Use `JoinSet` over manual `Vec<JoinHandle>` for dynamic task groups
- Reserve `spawn_blocking` strictly for blocking/CPU work

## Summary

`spawn` for concurrent async tasks (`Send + 'static`), `spawn_local` for `!Send`, `spawn_blocking` for CPU/blocking work. `JoinSet` manages task groups. Always move data into spawned tasks.

## Code Examples

**Using JoinSet to manage multiple concurrent tasks**

```rust
use tokio::task::JoinSet;

#[tokio::main]
async fn main() {
    let mut set = JoinSet::new();

    for i in 0..5 {
        set.spawn(async move {
            tokio::time::sleep(std::time::Duration::from_millis(100)).await;
            i * 2
        });
    }

    while let Some(result) = set.join_next().await {
        println!("Got: {:?}", result.unwrap());
    }
}
```


## Resources

- [Spawning](https://tokio.rs/tokio/tutorial/spawning) — Tokio tutorial on spawning tasks

---

> 📘 *This lesson is part of the [Asynchronous Rust](https://stanza.dev/courses/rust-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*