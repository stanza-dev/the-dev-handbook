---
source_course: "rust-async"
source_lesson: "rust-async-cancellation-safety"
---

# Understanding Cancellation

In Rust async, dropping a Future cancels it. This can cause issues:

```rust
async fn dangerous() {
    let mut data = fetch_initial().await;
    
    // If cancelled HERE, we have partial state!
    modify(&mut data);
    
    save(data).await;  // Never runs if cancelled above
}
```

## Cancellation-Safe Operations

An operation is **cancellation-safe** if cancelling and restarting produces correct results.

**Safe:**
- `tokio::time::sleep()` - Can restart from beginning
- `TcpStream::read()` - Data stays in buffer
- `channel::recv()` - Message not lost on cancel

**Unsafe:**
- `BufReader::read_line()` - Partial data lost!
- Custom operations with intermediate state

## Making Operations Safe

### Pattern 1: Use cancelation-safe alternatives

```rust
// Instead of read_line (unsafe)
let line = reader.read_line(&mut buf).await?;

// Use tokio's cancellation-safe version
let line = tokio::io::AsyncBufReadExt::read_line(&mut reader, &mut buf).await?;
```

### Pattern 2: Complete sub-operations atomically

```rust
async fn safe_update(db: &Database, id: u64) {
    // Fetch and modify in one go before any await
    let update = prepare_update(id);
    
    // Now safe to await
    db.execute(update).await?;
}
```

### Pattern 3: Use tokio::select! carefully

```rust
let msg = loop {
    select! {
        // recv() is cancellation-safe
        msg = rx.recv() => break msg,
        _ = timeout => return Err("timeout"),
    }
};
```

## Code Examples

**Safe select loop**

```rust
// Cancellation-safe message processing
use tokio::sync::mpsc;

async fn process_messages(mut rx: mpsc::Receiver<Message>) {
    loop {
        // This is safe: recv() doesn't lose messages on cancel
        tokio::select! {
            Some(msg) = rx.recv() => {
                // Process completely before next select
                handle_message(msg).await;
            }
            _ = tokio::signal::ctrl_c() => break,
        }
    }
}
```


---

> 📘 *This lesson is part of the [Asynchronous Rust](https://stanza.dev/courses/rust-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*