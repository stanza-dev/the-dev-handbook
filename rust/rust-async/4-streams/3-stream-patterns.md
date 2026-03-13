---
source_course: "rust-async"
source_lesson: "rust-async-stream-patterns"
---

# Practical Stream Patterns

## Introduction

Streams shine in real-world scenarios: paginating APIs, processing files line-by-line, handling WebSocket connections, and event sourcing. These patterns appear constantly in production Rust services.

## Key Concepts

**Pagination** — yield pages of results as a stream:
```rust
use async_stream::stream;

fn paginated(api: &Api) -> impl Stream<Item = Item> + '_ {
    stream! {
        let mut page = 1;
        loop {
            let items = api.fetch_page(page).await;
            if items.is_empty() { break; }
            for item in items { yield item; }
            page += 1;
        }
    }
}
```

## Real World Context

Every API with pagination, every file processor, every message queue consumer, every real-time feed is naturally modeled as a stream.

## Deep Dive

**File processing:**
```rust
use tokio::io::{AsyncBufReadExt, BufReader};
use tokio_stream::wrappers::LinesStream;

async fn process_file(path: &str) -> Result<(), Error> {
    let file = tokio::fs::File::open(path).await?;
    let mut lines = LinesStream::new(BufReader::new(file).lines());
    while let Some(line) = lines.next().await {
        process_line(&line?).await?;
    }
    Ok(())
}
```

**WebSocket messages:**
```rust
async fn handle_ws(ws: WebSocket) {
    let (mut tx, mut rx) = ws.split();
    while let Some(Ok(msg)) = rx.next().await {
        if let Message::Text(text) = msg {
            let response = process(&text).await;
            tx.send(Message::Text(response)).await.ok();
        }
    }
}
```

**Event sourcing with broadcast:**
```rust
let stream = BroadcastStream::new(event_rx);
stream
    .filter_map(|r| async { r.ok() })
    .for_each(|event| async { handle_event(event).await })
    .await;
```

## Common Pitfalls

- Not handling stream errors — use `filter_map` or explicit error handling
- Unbounded stream processing — add backpressure with `buffer_unordered`
- Creating streams that never end without a shutdown mechanism

## Best Practices

- Use `async_stream::stream!` for custom pagination and generation logic
- Wrap channel receivers with `ReceiverStream` or `BroadcastStream`
- Always plan for stream termination (channel close, EOF, shutdown signal)

## Summary

Streams model pagination, file I/O, WebSockets, and event sourcing naturally. Use `async_stream::stream!` for generators. Wrap channels with stream wrappers. Always handle errors and plan termination.

## Code Examples

**Cursor-based pagination exposed as a stream**

```rust
use async_stream::stream;
use tokio_stream::StreamExt;

// Paginated API consumer as a stream
fn fetch_all_users(client: &Client) -> impl Stream<Item = User> + '_ {
    stream! {
        let mut cursor = None;
        loop {
            let page = client.get_users(cursor).await.unwrap();
            cursor = page.next_cursor;
            for user in page.users {
                yield user;
            }
            if cursor.is_none() { break; }
        }
    }
}

async fn example(client: &Client) {
    let mut users = std::pin::pin!(fetch_all_users(client));
    while let Some(user) = users.next().await {
        println!("{}", user.name);
    }
}
```


## Resources

- [async-stream crate](https://docs.rs/async-stream/latest/async_stream/) — The async_stream macro crate documentation

---

> 📘 *This lesson is part of the [Asynchronous Rust](https://stanza.dev/courses/rust-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*