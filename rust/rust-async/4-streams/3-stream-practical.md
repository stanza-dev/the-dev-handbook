---
source_course: "rust-async"
source_lesson: "rust-async-stream-practical"
---

# Real-World Stream Patterns

## Pagination with Streams

```rust
use async_stream::stream;

fn paginated_fetch(api: &Api) -> impl Stream<Item = Result<Item, Error>> + '_ {
    stream! {
        let mut page = 1;
        loop {
            let response = api.fetch_page(page).await?;
            if response.items.is_empty() {
                break;
            }
            for item in response.items {
                yield Ok(item);
            }
            page += 1;
        }
    }
}
```

## File Processing

```rust
use tokio::io::{AsyncBufReadExt, BufReader};
use tokio_stream::wrappers::LinesStream;

async fn process_file(path: &str) -> Result<(), Error> {
    let file = tokio::fs::File::open(path).await?;
    let reader = BufReader::new(file);
    let mut lines = LinesStream::new(reader.lines());
    
    while let Some(line) = lines.next().await {
        let line = line?;
        process_line(&line).await?;
    }
    Ok(())
}
```

## WebSocket Messages

```rust
use futures::StreamExt;

async fn handle_websocket(ws: WebSocket) {
    let (mut tx, mut rx) = ws.split();
    
    while let Some(msg) = rx.next().await {
        match msg {
            Ok(Message::Text(text)) => {
                let response = process(&text).await;
                tx.send(Message::Text(response)).await?;
            }
            Ok(Message::Close(_)) => break,
            Err(e) => eprintln!("Error: {e}"),
            _ => {}
        }
    }
}
```

## Code Examples

**Event processing with streams**

```rust
// Event sourcing with streams
use tokio::sync::broadcast;

async fn event_processor(
    mut events: broadcast::Receiver<Event>
) {
    let stream = tokio_stream::wrappers::BroadcastStream::new(events);
    
    stream
        .filter_map(|r| async { r.ok() })
        .for_each(|event| async {
            match event {
                Event::Created(id) => handle_created(id).await,
                Event::Updated(id) => handle_updated(id).await,
                Event::Deleted(id) => handle_deleted(id).await,
            }
        })
        .await;
}
```


---

> 📘 *This lesson is part of the [Asynchronous Rust](https://stanza.dev/courses/rust-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*