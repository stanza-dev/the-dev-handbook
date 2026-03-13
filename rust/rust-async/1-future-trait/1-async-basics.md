---
source_course: "rust-async"
source_lesson: "rust-async-basics"
---

# Async/Await Fundamentals

## Introduction

Rust's `async`/`.await` lets you write non-blocking code that reads like synchronous code. The `async` keyword transforms a function into one that returns an `impl Future` instead of its declared return type. The `.await` keyword suspends execution until the future resolves.

Since the 2024 edition, `Future` and `IntoFuture` are in the prelude — no imports needed.

## Key Concepts

**The `async` keyword** transforms a function body into a state machine wrapped in a `Future`:

```rust
async fn fetch_data() -> String {
    // Returns impl Future<Output = String>, NOT String directly
    "data".to_string()
}
```

**The `.await` keyword** suspends the current future until the awaited one resolves:

```rust
async fn process() {
    let data = fetch_data().await; // Suspend here
    println!("Got: {data}");
}
```

**Async blocks** create anonymous futures that capture their environment:

```rust
let my_future = async {
    let x = fetch_one().await;
    let y = fetch_two().await;
    x + y
};
```

## Real World Context

Async Rust powers high-performance servers (Axum, Actix), database drivers (sqlx), and HTTP clients (reqwest). Any I/O-bound workload benefits from async.

## Deep Dive

Futures in Rust are **lazy** — nothing happens until you `.await` or poll them:

```rust
let future = fetch_data(); // Future created, NOT started
// ... other code ...
let data = future.await;   // NOW it executes
```

This is fundamentally different from JavaScript Promises or Go goroutines, which start immediately.\n\nSince Rust 1.85, **async closures** provide a first-class way to create closures that return futures:\n\n```rust\n// async closure — can borrow from captures (unlike || async {})\nlet name = String::from(\"Alice\");\nlet greet = async || {\n    println!(\"Hello, {name}\"); // borrows name\n};\ngreet().await;\n```\n\nThe `AsyncFn`, `AsyncFnMut`, and `AsyncFnOnce` traits (in the 2024 edition prelude) are the async equivalents of `Fn`, `FnMut`, and `FnOnce`.

## Common Pitfalls

- Forgetting `.await` — the future never executes and the compiler warns about unused `Future`
- Assuming futures run in the background once created — they don't
- Mixing sync and async code without `spawn_blocking`

## Best Practices

- Always `.await` or explicitly spawn futures you create
- Keep async functions focused on I/O; offload CPU work to blocking threads
- Use `async move` blocks when you need to transfer ownership into the future
- Since Rust 1.85, use `async closures` (`async || {}`) when you need closures that return futures and borrow from captures — the `AsyncFn`, `AsyncFnMut`, and `AsyncFnOnce` traits power this

## Summary

`async` creates futures, `.await` drives them. Futures are lazy and do nothing until polled. The 2024 edition puts `Future` in the prelude. Async blocks capture their environment like closures. Since Rust 1.85, async closures (`async || {}`) provide native support for closures that return futures.

## Code Examples

**Lazy evaluation of futures — nothing executes until .await**

```rust
// Futures are lazy — nothing runs until awaited
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

- [Async Book](https://rust-lang.github.io/async-book/) — The official async programming in Rust book

---

> 📘 *This lesson is part of the [Asynchronous Rust](https://stanza.dev/courses/rust-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*