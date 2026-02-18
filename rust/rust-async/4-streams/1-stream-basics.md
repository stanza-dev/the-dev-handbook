---
source_course: "rust-async"
source_lesson: "rust-async-stream-basics"
---

# Streams: Async Iterators

A `Stream` yields multiple values asynchronously - like an async Iterator.

```rust
pub trait Stream {
    type Item;
    fn poll_next(
        self: Pin<&mut Self>,
        cx: &mut Context<'_>
    ) -> Poll<Option<Self::Item>>;
}
```

| Sync | Async |
|------|-------|
| `Iterator` | `Stream` |
| `next() -> Option<T>` | `poll_next() -> Poll<Option<T>>` |
| `for item in iter` | `while let Some(item) = stream.next().await` |

## Using Streams

```rust
use tokio_stream::StreamExt;

let mut stream = tokio_stream::iter(vec![1, 2, 3]);

while let Some(item) = stream.next().await {
    println!("{item}");
}
```

## Creating Streams

### From iterators:
```rust
use tokio_stream::StreamExt;

let stream = tokio_stream::iter(1..=5);
```

### With async_stream macro:
```rust
use async_stream::stream;

let s = stream! {
    for i in 0..3 {
        tokio::time::sleep(Duration::from_millis(100)).await;
        yield i;
    }
};
```

### From channels:
```rust
use tokio::sync::mpsc;
use tokio_stream::wrappers::ReceiverStream;

let (tx, rx) = mpsc::channel(32);
let stream = ReceiverStream::new(rx);
```

See [Streams](https://tokio.rs/tokio/tutorial/streams).

## Code Examples

**Basic stream usage**

```rust
use tokio_stream::StreamExt;

#[tokio::main]
async fn main() {
    // Create a stream that yields values with delay
    let stream = tokio_stream::iter(1..=5)
        .throttle(Duration::from_millis(100));
    
    tokio::pin!(stream);
    
    while let Some(value) = stream.next().await {
        println!("Got: {value}");
    }
}
```


---

> 📘 *This lesson is part of the [Asynchronous Rust](https://stanza.dev/courses/rust-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*