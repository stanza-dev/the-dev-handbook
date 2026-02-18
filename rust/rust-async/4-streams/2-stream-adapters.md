---
source_course: "rust-async"
source_lesson: "rust-async-stream-adapters"
---

# StreamExt Adapters

The `StreamExt` trait provides combinator methods similar to Iterator.

## Common Adapters

| Adapter | Description |
|---------|-------------|
| `map` | Transform each item |
| `filter` | Keep items matching predicate |
| `filter_map` | Filter and transform |
| `take` | Take first N items |
| `skip` | Skip first N items |
| `take_while` | Take while predicate is true |
| `merge` | Interleave two streams |
| `chain` | Concatenate streams |
| `throttle` | Rate limit items |
| `timeout` | Timeout per item |

```rust
use tokio_stream::StreamExt;

let result: Vec<_> = stream
    .filter(|x| *x % 2 == 0)
    .map(|x| x * 10)
    .take(5)
    .collect()
    .await;
```

## Buffered Execution

```rust
use futures::stream::StreamExt;

// Process items concurrently (up to 10 at a time)
stream
    .map(|item| async move { process(item).await })
    .buffer_unordered(10)  // Concurrent with no order guarantee
    .for_each(|result| async { handle(result) })
    .await;

// Or with ordering preserved:
stream
    .map(|item| async move { process(item).await })
    .buffered(10)  // Concurrent but ordered
    .collect::<Vec<_>>()
    .await;
```

## Merging Streams

```rust
use tokio_stream::StreamExt;

let combined = stream1.merge(stream2);
// Items interleaved from both streams
```

## Code Examples

**Concurrent HTTP requests**

```rust
use futures::stream::{self, StreamExt};

async fn process_urls(urls: Vec<String>) -> Vec<Response> {
    stream::iter(urls)
        .map(|url| async move {
            reqwest::get(&url).await
        })
        .buffer_unordered(10) // 10 concurrent requests
        .filter_map(|r| async { r.ok() })
        .collect()
        .await
}
```


---

> 📘 *This lesson is part of the [Asynchronous Rust](https://stanza.dev/courses/rust-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*