---
source_course: "rust-async"
source_lesson: "rust-async-async-basics"
---

# Async/Await Fundamentals

Async programming in Rust allows writing non-blocking code that looks synchronous.

## The async Keyword

```rust
async fn fetch_data() -> String {
    // This returns impl Future<Output = String>, not String!
    "data".to_string()
}
```

The `async` keyword transforms the function into one that returns a `Future`. The function body becomes the state machine that the Future will execute.

## The await Keyword

```rust
async fn process() {
    let data = fetch_data().await; // Suspends until complete
    println!("Got: {data}");
}
```

**Critical insight:** Nothing happens until you `.await` or poll the Future!

```rust
let future = fetch_data(); // Future created but NOT started
// ... other code ...
let data = future.await;   // NOW it executes
```

## What async/await Compiles To

```rust
// This:
async fn foo() -> i32 { 42 }

// Roughly becomes:
fn foo() -> impl Future<Output = i32> {
    async { 42 }
}
```

## Async Blocks

```rust
let my_future = async {
    let x = fetch_one().await;
    let y = fetch_two().await;
    x + y
};
```

Async blocks capture their environment and create anonymous Future types.

See [Async Programming in Rust](https://rust-lang.github.io/async-book/).

## Code Examples

**Lazy evaluation of futures**

```rust
// Futures are lazy!
async fn expensive() -> i32 {
    println!("Computing..."); // Only prints when awaited
    42
}

async fn main_async() {
    let future = expensive(); // Nothing printed yet!
    println!("Created future");
    let result = future.await; // NOW "Computing..." prints
    println!("Result: {result}");
}
```


## Resources

- [Async Book](https://rust-lang.github.io/async-book/) — The official async Rust book

---

> 📘 *This lesson is part of the [Asynchronous Rust](https://stanza.dev/courses/rust-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*