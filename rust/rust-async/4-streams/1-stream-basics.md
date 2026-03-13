---
source_course: "rust-async"
source_lesson: "rust-async-stream-basics"
---

# The Stream Trait

## Introduction

A `Stream` yields multiple values asynchronously — it's the async version of `Iterator`. While `Iterator::next()` returns `Option<T>` synchronously, `Stream::poll_next()` returns `Poll<Option<T>>` and can suspend between items. Note: `for await` syntax is still nightly-only.

## Key Concepts

```rust
pub trait Stream {
    type Item;
    fn poll_next(
        self: Pin<&mut Self>,
        cx: &mut Context<'_>,
    ) -> Poll<Option<Self::Item>>;
}
```

| Sync | Async |
|------|-------|
| `Iterator` | `Stream` |
| `next() -> Option<T>` | `poll_next() -> Poll<Option<T>>` |
| `for item in iter` | `while let Some(item) = stream.next().await` |

## Real World Context

Streams model WebSocket messages, database cursors, paginated API responses, file lines, Kafka topics — any sequence of values arriving over time.

## Deep Dive

**Using streams** with `StreamExt::next()`:
```rust
use tokio_stream::StreamExt;

let mut stream = tokio_stream::iter(vec![1, 2, 3]);
while let Some(item) = stream.next().await {
    println!("{item}");
}
```

**Creating streams from iterators:**
```rust
let stream = tokio_stream::iter(1..=5);
```

**With `async_stream` macro:**
```rust
use async_stream::stream;
let s = stream! {
    for i in 0..3 {
        tokio::time::sleep(Duration::from_millis(100)).await;
        yield i;
    }
};
```

**From channels:**
```rust
let (tx, rx) = tokio::sync::mpsc::channel(32);
let stream = tokio_stream::wrappers::ReceiverStream::new(rx);
```

## Common Pitfalls

- Using `for` loops on streams — you need `while let Some(x) = stream.next().await` (or nightly `for await`)
- Forgetting to pin streams — `tokio::pin!(stream)` is often needed
- Not importing `StreamExt` — the `.next()` method comes from this trait

## Best Practices

- Import `StreamExt` for ergonomic stream consumption
- Use `async_stream::stream!` for custom stream logic
- Pin streams with `tokio::pin!` when needed by adapters

## Summary

Streams are async iterators yielding `Poll<Option<T>>`. Use `StreamExt::next().await` to consume them. Create streams from iterators, channels, or the `async_stream` macro. Standard `for` loops don't work — use `while let`.

## Code Examples

**Basic stream consumption with throttling**

```rust
use tokio_stream::StreamExt;
use std::time::Duration;

#[tokio::main]
async fn main() {
    let stream = tokio_stream::iter(1..=5)
        .throttle(Duration::from_millis(200));

    tokio::pin!(stream);

    while let Some(value) = stream.next().await {
        println!("Got: {value}");
    }
}
```


## Resources

- [Tokio Streams](https://tokio.rs/tokio/tutorial/streams) — Tokio tutorial on streams

---

> 📘 *This lesson is part of the [Asynchronous Rust](https://stanza.dev/courses/rust-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*