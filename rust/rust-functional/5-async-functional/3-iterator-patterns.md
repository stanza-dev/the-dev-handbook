---
source_course: "rust-functional"
source_lesson: "rust-func-async-iterator-patterns"
---

# Async Iterator Patterns

## Introduction
Async iterators (also called streams) yield values asynchronously — each element may require awaiting an I/O operation, a timer, or another async task. Combined with async closures, they enable functional-style data processing over asynchronous data sources like network connections, message queues, and file streams.

## Key Concepts
- **`Stream` trait**: The async equivalent of `Iterator`. Defined in `futures`/`tokio-stream` as `trait Stream { fn poll_next(...) -> Poll<Option<Self::Item>>; }`.
- **`async-iterator` (nightly)**: The standard library is working on `AsyncIterator` as the official async iterator trait. For stable Rust, use the `futures::Stream` trait.
- **Stream combinators**: Async versions of `map`, `filter`, `fold`, and others that apply async transformations to streams.

## Real World Context
Async iterators power real-time data processing: reading lines from a TCP socket, processing messages from a Kafka consumer, watching filesystem changes, or streaming database query results row by row. They combine the efficiency of async I/O with the expressiveness of iterator chains.

## Deep Dive

### Creating a stream from an async closure

The `futures::stream` module provides utilities to create streams:

```rust
use futures::stream::{self, StreamExt};

// Create a stream that yields values from an async computation
let page_stream = stream::unfold(1u32, |page| async move {
    if page > 5 {
        None // Stop after 5 pages
    } else {
        let data = fetch_page(page).await;
        Some((data, page + 1)) // (yielded_value, next_state)
    }
});
```

`unfold` is the async equivalent of building an iterator with a state machine. The async closure receives the current state and returns `Option<(Item, NextState)>`.

### Stream combinators with async closures

Stream combinators work like iterator adapters but support async operations:

```rust
use futures::stream::{self, StreamExt};

let user_ids = stream::iter(vec![1, 2, 3, 4, 5]);

// map with an async operation
let users = user_ids.then(|id| async move {
    fetch_user(id).await
});

// filter with an async predicate
let active_users = users.filter(|user| async {
    user.is_active().await
});

// collect all results
let result: Vec<User> = active_users.collect().await;
```

The `.then()` method applies an async function to each element, and `.filter()` accepts an async predicate.

### Buffered concurrent processing

Streams support concurrent processing with backpressure control:

```rust
use futures::stream::{self, StreamExt};

let urls = stream::iter(vec![
    "https://api.example.com/users",
    "https://api.example.com/posts",
    "https://api.example.com/comments",
]);

// Process up to 3 requests concurrently
let responses: Vec<Response> = urls
    .map(|url| async move {
        reqwest::get(url).await.unwrap()
    })
    .buffer_unordered(3) // Max 3 concurrent futures
    .collect()
    .await;
```

`buffer_unordered(n)` polls up to `n` futures concurrently, yielding results as they complete (not necessarily in order). Use `buffered(n)` for ordered results.

### Combining async closures with streams

With Rust 1.85's async closures, you can write cleaner stream transformations:

```rust
use futures::stream::StreamExt;

// Create a processing pipeline using async closures
let process_event = async |event: Event| -> ProcessedEvent {
    let enriched = enrich_event(event).await;
    validate_event(enriched).await
};

let results: Vec<ProcessedEvent> = event_stream
    .then(process_event)
    .collect()
    .await;
```

The async closure captures the processing logic cleanly, and `.then()` applies it to each stream element.

## Common Pitfalls
1. **Forgetting `.await` on stream consumers** — `stream.collect()` returns a future. You must `.await` it to drive the stream.
2. **Unbounded concurrency** — Using `buffer_unordered` without a limit can overwhelm downstream systems. Always specify a reasonable concurrency limit.
3. **Mixing `Iterator` and `Stream`** — `stream::iter()` converts a sync iterator into a stream. Do not confuse `Iterator::map` with `StreamExt::map`.

## Best Practices
1. **Use `buffer_unordered` for I/O-bound work** — It maximizes throughput by processing multiple items concurrently.
2. **Prefer `stream::iter` for known collections** — When the source data is already in memory, `stream::iter` is the simplest way to create a stream.
3. **Handle errors in the stream** — Use `.filter_map()` or `.try_for_each()` to handle errors per-element rather than panicking.

## Summary
- Async iterators (streams) yield values asynchronously.
- `futures::stream` provides stream creation and combinator utilities.
- `.then()` applies an async function to each stream element.
- `buffer_unordered(n)` enables bounded concurrent processing.
- Async closures (Rust 1.85+) make stream transformations concise and readable.

## Code Examples

**A complete async data pipeline — paginated fetching, concurrent detail lookups with backpressure, and filtering — all composed functionally**

```rust
use futures::stream::{self, StreamExt};

// A practical async data pipeline:
// 1. Fetch user IDs from a paginated API
// 2. Fetch each user's details concurrently (max 5 at a time)
// 3. Filter to active users
// 4. Collect results
async fn get_active_users() -> Vec<User> {
    let user_ids = stream::unfold(0u32, |page| async move {
        let ids = fetch_user_ids_page(page).await;
        if ids.is_empty() {
            None
        } else {
            Some((stream::iter(ids), page + 1))
        }
    })
    .flatten(); // Flatten stream of streams into single stream

    user_ids
        .map(|id| async move { fetch_user_detail(id).await })
        .buffer_unordered(5) // Fetch 5 users concurrently
        .filter(|user| futures::future::ready(user.is_active))
        .collect()
        .await
}

// Output: Vec of all active users, fetched efficiently with
// controlled concurrency and functional composition
```


## Resources

- [futures::stream module documentation](https://docs.rs/futures/latest/futures/stream/index.html) — API reference for the futures stream module with stream creation utilities and combinators

---

> 📘 *This lesson is part of the [Functional Rust](https://stanza.dev/courses/rust-functional) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*