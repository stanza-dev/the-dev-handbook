---
source_course: "rust-async"
source_lesson: "rust-async-cancellation"
---

# Cancellation Safety

## Introduction

In Rust async, dropping a future cancels it mid-execution. If the future was partway through a multi-step operation, you can end up with partial state. Understanding which operations are cancellation-safe is critical when using `select!`.

## Key Concepts

An operation is **cancellation-safe** if dropping and restarting it produces correct results — no data is lost or corrupted.

**Safe operations:**
- `tokio::time::sleep` — restarting just sleeps again
- `mpsc::Receiver::recv` — message stays in channel if cancelled
- `TcpStream::read` — unread data stays in the kernel buffer

**Unsafe operations:**
- `BufReader::read_line` — partial data already read into buffer is lost
- Multi-step state mutations with awaits between them

## Real World Context

Every `select!` branch can be cancelled at any `.await` point. If your branch reads a message, modifies state, then writes — cancellation between steps means lost messages or inconsistent state.

## Deep Dive

**Pattern 1: Use cancellation-safe alternatives**
```rust
// BufReader::read_line is NOT safe in select!
// Use tokio::io::AsyncBufReadExt or process complete messages
```

**Pattern 2: Atomic sub-operations**
```rust
async fn safe_update(db: &Database, id: u64) -> Result<()> {
    let update = prepare_update(id); // No await — all sync
    db.execute(update).await?;       // Single await, atomic
    Ok(())
}
```

**Pattern 3: Pre-read in select**
```rust
let msg = loop {
    select! {
        msg = rx.recv() => break msg, // recv is safe
        _ = timeout => return Err("timeout"),
    }
};
// Process msg outside select — no cancellation risk
```

## Common Pitfalls

- Assuming all async operations are cancellation-safe
- Doing multi-step mutations inside `select!` branches
- Ignoring Tokio's cancellation safety docs for each method

## Best Practices

- Check Tokio docs — each method documents its cancellation safety
- Keep select branches minimal; do heavy work after the select
- Use `tokio::pin!` and re-poll the same future across loop iterations

## Summary

Cancellation-safe means drop-and-restart is correct. `recv`, `sleep`, and raw I/O reads are safe. `BufReader::read_line` and multi-step mutations are not. Keep select branches lean and do processing outside the select.

## Code Examples

**Cancellation-safe select loop using mpsc::recv**

```rust
use tokio::sync::mpsc;

// Safe: recv() doesn't lose messages on cancellation
async fn safe_loop(mut rx: mpsc::Receiver<String>) {
    loop {
        tokio::select! {
            Some(msg) = rx.recv() => {
                // Process completely before next select
                handle_message(msg).await;
            }
            _ = tokio::signal::ctrl_c() => {
                println!("Shutting down");
                break;
            }
        }
    }
}
```


## Resources

- [Cancellation Safety](https://docs.rs/tokio/latest/tokio/macro.select.html#cancellation-safety) — Tokio docs on cancellation safety in select!

---

> 📘 *This lesson is part of the [Asynchronous Rust](https://stanza.dev/courses/rust-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*