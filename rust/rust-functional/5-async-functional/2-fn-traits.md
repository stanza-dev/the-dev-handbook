---
source_course: "rust-functional"
source_lesson: "rust-func-async-fn-traits"
---

# AsyncFn Traits and Bounds

## Introduction
Just as synchronous closures implement `Fn`, `FnMut`, or `FnOnce`, async closures implement `AsyncFn`, `AsyncFnMut`, or `AsyncFnOnce`. These traits were stabilized in Rust 1.85 and added to the prelude, so they are available without any imports. Understanding these bounds is essential for writing generic async APIs.

## Key Concepts
- **`AsyncFn`**: An async closure that only borrows captures immutably. Can be called multiple times concurrently.
- **`AsyncFnMut`**: An async closure that may mutate captures. Can be called multiple times but not concurrently.
- **`AsyncFnOnce`**: An async closure that may consume captures. Can be called at most once.
- **Trait hierarchy**: `AsyncFn` ⊂ `AsyncFnMut` ⊂ `AsyncFnOnce`, mirroring the sync hierarchy.

## Real World Context
Web frameworks need to accept async handlers generically. Axum's handler system, for instance, requires closures that can be called for each request. With `AsyncFn` bounds, you can write middleware and handler registration APIs that accept async closures directly, without requiring users to wrap their logic in boxed futures.

## Deep Dive

### Using AsyncFn bounds

To accept an async closure in a generic function, use `AsyncFn` bounds:

```rust
// Accepts any async closure that takes a &str and returns a String
async fn process_with<F>(handler: F, input: &str) -> String
where
    F: AsyncFn(&str) -> String,
{
    handler(input).await
}

// Usage
let result = process_with(
    async |name: &str| format!("Hello, {name}!"),
    "Alice",
).await;
assert_eq!(result, "Hello, Alice!");
```

The bound `AsyncFn(&str) -> String` means: "a callable that takes `&str`, returns a `Future<Output = String>`, and only borrows captures immutably."

### AsyncFnMut for stateful handlers

When the closure needs to mutate state between calls:

```rust
async fn call_n_times<F>(mut handler: F, times: u32)
where
    F: AsyncFnMut() -> (),
{
    for _ in 0..times {
        handler().await;
    }
}

let mut counter = 0;
call_n_times(async || {
    counter += 1;
    println!("Call #{counter}");
}, 3).await;
```

Note the `mut handler` parameter — just like `FnMut`, you need mutable access to call an `AsyncFnMut`.

### AsyncFnOnce for consuming operations

Some operations should only happen once — database migrations, one-time initialization:

```rust
async fn run_once<F, T>(setup: F) -> T
where
    F: AsyncFnOnce() -> T,
{
    setup().await
}

let db_pool = create_pool().await;
let migration_result = run_once(async || {
    // Consumes db_pool — can only run once
    run_migrations(db_pool).await
}).await;
```

### Comparing with the old approach

Before AsyncFn traits, you had to use complex bounds:

```rust
// Old approach (pre-1.85)
async fn old_process<F, Fut>(handler: F, input: String) -> String
where
    F: Fn(String) -> Fut,
    Fut: Future<Output = String>,
{
    handler(input).await
}

// New approach (1.85+)
async fn new_process<F>(handler: F, input: String) -> String
where
    F: AsyncFn(String) -> String,
{
    handler(input).await
}
```

The new syntax is more concise and correctly handles the relationship between the closure and its returned future.

## Common Pitfalls
1. **Using `AsyncFn` when `AsyncFnOnce` suffices** — If you only call the closure once, use `AsyncFnOnce` for maximum flexibility. `AsyncFn` unnecessarily restricts to immutable captures.
2. **Trying to call `AsyncFnMut` concurrently** — Unlike `AsyncFn`, `AsyncFnMut` closures cannot be called concurrently because they require mutable access.
3. **Mixing old and new syntax** — Avoid using `Fn() -> impl Future<Output = T>` bounds when `AsyncFn() -> T` is available. The new bounds handle lifetimes more correctly.

## Best Practices
1. **Use the weakest bound needed** — `AsyncFnOnce` > `AsyncFnMut` > `AsyncFn` in terms of flexibility for callers.
2. **Prefer `impl AsyncFn` in argument position** — For simple cases, `handler: impl AsyncFn(Args) -> Output` reads cleanly.
3. **Migrate old `Fn() -> Future` bounds** — When updating to Rust 1.85+, replace two-generic patterns with single `AsyncFn` bounds.

## Summary
- `AsyncFn`, `AsyncFnMut`, `AsyncFnOnce` are the async equivalents of `Fn`, `FnMut`, `FnOnce`.
- They are in the prelude — no imports needed.
- Use the weakest bound (`AsyncFnOnce`) unless you need repeated calls.
- The new traits simplify generic async APIs by eliminating the `F: Fn() -> Fut, Fut: Future` pattern.
- The hierarchy mirrors sync closures: `AsyncFn` ⊂ `AsyncFnMut` ⊂ `AsyncFnOnce`.

## Code Examples

**AsyncFn-based middleware for timing async operations — shows how the new trait bounds simplify generic async code**

```rust
// Middleware pattern using AsyncFn
async fn timed_execution<F, T>(
    label: &str,
    handler: F,
) -> T
where
    F: AsyncFn() -> T,
{
    let start = std::time::Instant::now();
    let result = handler().await;
    let elapsed = start.elapsed();
    println!("{label} completed in {elapsed:?}");
    result
}

// Old vs New comparison
// Old: F: Fn(Request) -> Fut, Fut: Future<Output = Response>
// New: F: AsyncFn(Request) -> Response

async fn handle_request<F>(handler: F, request: Request) -> Response
where
    F: AsyncFn(Request) -> Response,
{
    timed_execution("Request", async || handler(request).await).await
}
```


## Resources

- [AsyncFn trait documentation](https://doc.rust-lang.org/std/ops/trait.AsyncFn.html) — API reference for the AsyncFn trait with signature details and usage examples

---

> 📘 *This lesson is part of the [Functional Rust](https://stanza.dev/courses/rust-functional) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*