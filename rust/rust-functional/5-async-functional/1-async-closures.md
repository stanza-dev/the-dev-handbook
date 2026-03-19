---
source_course: "rust-functional"
source_lesson: "rust-func-async-closures"
---

# Async Closures in Rust

## Introduction
Rust 1.85 stabilized `async` closures — closures that return a `Future` and can be called with `.await`. Before this, creating async closures required verbose workarounds with `async move` blocks. Now you can write `async || { ... }` directly, making functional async code as natural as synchronous code.

## Key Concepts
- **Async closure**: A closure defined with `async || { ... }` that returns a `Future` when called. The body is not executed until the returned future is awaited.
- **`AsyncFn` / `AsyncFnMut` / `AsyncFnOnce`**: The async equivalents of `Fn`, `FnMut`, and `FnOnce`. These traits are now in the prelude as of Rust 1.85.
- **Async block workaround**: The pre-1.85 pattern of `|| async { ... }` which had subtle borrowing limitations.

## Real World Context
Async closures simplify callback-heavy async code. HTTP middleware, event handlers, retry logic, and stream transformations all benefit from first-class async closures. Libraries like `tower` (used by Axum) and `tokio` can now accept async closures directly.

## Deep Dive

### The old way: closure returning async block

Before Rust 1.85, you had to write a closure that returns an async block:

```rust
// Pre-1.85: closure returning an async block
let fetch_data = |url: String| async move {
    reqwest::get(&url).await?.text().await
};
```

This worked for many cases, but had a critical limitation: the closure and the async block had separate capture environments. Borrowing data in the async block that was captured by the closure often led to lifetime errors.

### The new way: async closures

Rust 1.85 introduced native async closure syntax:

```rust
// Rust 1.85+: native async closure
let fetch_data = async |url: String| {
    reqwest::get(&url).await?.text().await
};
```

The `async` keyword before the parameter list makes the entire closure async. The closure body can use `.await` directly, and captures work correctly with the closure's lifetime.

### Calling async closures

Async closures return a `Future` that must be awaited:

```rust
let greet = async |name: &str| -> String {
    // Simulate async work
    tokio::time::sleep(std::time::Duration::from_millis(10)).await;
    format!("Hello, {name}!")
};

// Call the closure — returns a Future
let future = greet("Alice");

// Await the future to get the result
let message = future.await;
assert_eq!(message, "Hello, Alice!");
```

The closure itself is synchronous to call (it returns immediately with a `Future`), but the returned future is asynchronous.

### Async closures with mutable state

Async closures can capture and mutate state, just like synchronous closures:

```rust
let mut attempt_count = 0;

let retry_fetch = async |url: &str| {
    attempt_count += 1;
    println!("Attempt #{attempt_count} for {url}");
    reqwest::get(url).await
};

// Each call increments the counter
let _ = retry_fetch("https://api.example.com/data").await;
let _ = retry_fetch("https://api.example.com/data").await;
// Prints: Attempt #1, Attempt #2
```

This closure implements `AsyncFnMut` because it mutates `attempt_count`.

## Common Pitfalls
1. **Confusing `async || {}` with `|| async {}`** — The native `async || {}` form has proper capture semantics. The old `|| async {}` form creates a separate capture scope for the async block, causing lifetime issues with borrows.
2. **Forgetting to await** — Calling an async closure returns a `Future`, not the result. If you forget `.await`, you get the future object instead of the computed value.
3. **Edition requirement** — Async closures require Rust 1.85+ with the 2021 edition. They will not compile on older toolchains.

## Best Practices
1. **Prefer `async ||` over `|| async move`** — The native syntax handles borrows correctly and is more concise.
2. **Use `AsyncFn` bounds in generic functions** — When accepting an async closure parameter, use `impl AsyncFn(Args) -> Output` bounds.
3. **Combine with `.await` chains** — Async closures compose naturally with other async operations and `.await` chains.

## Summary
- Rust 1.85 stabilized `async || { ... }` closure syntax.
- Async closures return a Future that must be awaited.
- They fix the capture/borrowing issues of the old `|| async { }` pattern.
- `AsyncFn`, `AsyncFnMut`, `AsyncFnOnce` traits mirror the sync Fn hierarchy.
- Use `async ||` for middleware, retry logic, and stream transformations.

## Code Examples

**A generic retry function that accepts an AsyncFnMut closure — demonstrates how async closures compose with generic async utilities**

```rust
// Async closure for retry logic
async fn with_retry<F, T, E>(
    max_attempts: u32,
    mut operation: F,
) -> Result<T, E>
where
    F: AsyncFnMut() -> Result<T, E>,
{
    let mut last_error = None;
    for attempt in 1..=max_attempts {
        match operation().await {
            Ok(value) => return Ok(value),
            Err(e) => {
                println!("Attempt {attempt} failed, retrying...");
                last_error = Some(e);
            }
        }
    }
    Err(last_error.unwrap())
}

// Usage with an async closure
async fn example() {
    let result = with_retry(3, async || {
        fetch_remote_config().await
    }).await;
}
```


## Resources

- [Rust 1.85 Release Notes — Async Closures](https://blog.rust-lang.org/2025/02/20/Rust-1.85.0.html) — Official release announcement covering async closure stabilization and AsyncFn traits

---

> 📘 *This lesson is part of the [Functional Rust](https://stanza.dev/courses/rust-functional) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*