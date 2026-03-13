---
source_course: "rust-async"
source_lesson: "rust-async-stream-adapters"
---

# Stream Adapters & Combinators

## Introduction

Like iterators, streams have a rich set of adapters via `StreamExt`. These let you transform, filter, combine, and control the flow of async data without manual state management.

## Key Concepts

| Adapter | Description |
|---------|-------------|
| `map` | Transform each item |
| `filter` | Keep items matching a predicate |
| `take` / `skip` | Limit items |
| `merge` | Interleave two streams |
| `chain` | Concatenate two streams sequentially |
| `throttle` | Rate-limit item emission |
| `buffer_unordered` | Process N items concurrently, any order |
| `buffered` | Process N items concurrently, preserve order |

## Real World Context

Processing a stream of HTTP requests with bounded concurrency (`buffer_unordered`), merging WebSocket messages from multiple connections (`merge`), rate-limiting API calls (`throttle`).

## Deep Dive

**Chaining adapters:**
```rust
use tokio_stream::StreamExt;

let result: Vec<_> = stream
    .filter(|x| *x % 2 == 0)
    .map(|x| x * 10)
    .take(5)
    .collect()
    .await;
```

**Buffered concurrent processing:**
```rust
use futures::stream::StreamExt;

stream
    .map(|item| async move { process(item).await })
    .buffer_unordered(10)  // Up to 10 concurrent, any order
    .for_each(|result| async { handle(result) })
    .await;
```

**`buffered` vs `buffer_unordered`:**
- `buffered(n)` — concurrent but results come out in input order
- `buffer_unordered(n)` — concurrent, results come out as they finish

**Merging streams:**
```rust
let combined = stream_a.merge(stream_b);
// Items interleaved from both streams as they arrive
```

## Common Pitfalls

- `buffer_unordered(1000)` on network requests — overwhelms the server
- Confusing `buffered` (ordered) with `buffer_unordered` (unordered)
- Forgetting that `chain` is sequential — second stream starts only after first ends

## Best Practices

- Use `buffer_unordered` with a reasonable concurrency limit (10-100)
- Prefer `buffer_unordered` over `buffered` unless ordering matters
- Use `throttle` for rate limiting external API calls

## Summary

`StreamExt` provides `map`, `filter`, `take`, `merge`, `chain`, `throttle`, `buffered`, and `buffer_unordered`. Use `buffer_unordered(n)` for bounded concurrent processing. Adapters compose cleanly like iterator chains.

## Code Examples

**Bounded concurrent HTTP requests using buffer_unordered**

```rust
use futures::stream::{self, StreamExt};

async fn fetch_all(urls: Vec<String>) -> Vec<String> {
    stream::iter(urls)
        .map(|url| async move {
            reqwest::get(&url).await?.text().await
        })
        .buffer_unordered(10) // 10 concurrent requests max
        .filter_map(|r| async { r.ok() })
        .collect()
        .await
}
```


## Resources

- [StreamExt](https://docs.rs/futures/latest/futures/stream/trait.StreamExt.html) — Full StreamExt API reference

---

> 📘 *This lesson is part of the [Asynchronous Rust](https://stanza.dev/courses/rust-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*