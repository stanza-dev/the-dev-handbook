---
source_course: "rust-async"
source_lesson: "rust-async-tokio-spawn"
---

# Tasks: Lightweight Concurrent Units

A **task** is the async equivalent of a thread - much lighter!

```rust
#[tokio::main]
async fn main() {
    let handle = tokio::spawn(async {
        // Runs concurrently with main
        expensive_computation().await
    });
    
    // Do other work while task runs...
    
    let result = handle.await.unwrap();
}
```

## Task Requirements

Spawned tasks must be:

1. **`'static`** - Cannot borrow from the spawning function
2. **`Send`** - Can be moved between threads

```rust
let data = String::from("hello");

// Must MOVE ownership into task
tokio::spawn(async move {
    println!("{data}"); // data is moved in
});
// data no longer accessible here
```

## JoinHandle

```rust
let handle: JoinHandle<i32> = tokio::spawn(async { 42 });

match handle.await {
    Ok(value) => println!("Got: {value}"),
    Err(e) => println!("Task panicked: {e}"),
}
```

## spawn_local for !Send Futures

```rust
use tokio::task::LocalSet;

let local = LocalSet::new();

local.run_until(async {
    tokio::task::spawn_local(async {
        // Can use !Send types here
        let rc = std::rc::Rc::new(42);
        println!("{rc}");
    }).await.unwrap();
}).await;
```

## spawn_blocking for CPU-Bound Work

```rust
let result = tokio::task::spawn_blocking(|| {
    // Runs on dedicated blocking thread pool
    heavy_computation()
}).await?;
```

See [Spawning](https://tokio.rs/tokio/tutorial/spawning).

## Code Examples

**JoinSet for multiple tasks**

```rust
use tokio::task::JoinSet;

#[tokio::main]
async fn main() {
    let mut set = JoinSet::new();
    
    for i in 0..10 {
        set.spawn(async move {
            tokio::time::sleep(std::time::Duration::from_millis(100)).await;
            i * 2
        });
    }
    
    // Collect results as they complete
    while let Some(result) = set.join_next().await {
        println!("Got: {:?}", result.unwrap());
    }
}
```


---

> 📘 *This lesson is part of the [Asynchronous Rust](https://stanza.dev/courses/rust-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*