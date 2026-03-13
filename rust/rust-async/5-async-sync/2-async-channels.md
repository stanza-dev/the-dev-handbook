---
source_course: "rust-async"
source_lesson: "rust-async-channels"
---

# Async Channels Deep Dive

## Introduction

Tokio provides four channel types for async communication: `mpsc`, `oneshot`, `broadcast`, and `watch`. Each solves a different messaging pattern. Combined with the actor pattern, channels become the backbone of concurrent application architecture.

## Key Concepts

| Channel | Producers | Consumers | Buffer | Use Case |
|---------|-----------|-----------|--------|----------|
| `mpsc` | Many | One | Bounded | Task communication |
| `oneshot` | One | One | None | Request-response |
| `broadcast` | One+ | Many | Ring buffer | Pub/sub |
| `watch` | One | Many | Latest only | Config/state updates |

## Real World Context

Actor systems, command buses, configuration hot-reload, fan-out notification systems, and request-response patterns for background workers.

## Deep Dive

**mpsc** — bounded, backpressure-aware:
```rust
let (tx, mut rx) = tokio::sync::mpsc::channel::<String>(100);
tx.send("hello".into()).await.unwrap();
let msg = rx.recv().await; // Some("hello")
```

**oneshot** — single value, single use:
```rust
let (tx, rx) = tokio::sync::oneshot::channel();
tx.send(42).unwrap(); // No await needed
let val = rx.await.unwrap(); // 42
```

**broadcast** — all subscribers get every message:
```rust
let (tx, _) = tokio::sync::broadcast::channel(16);
let mut rx1 = tx.subscribe();
let mut rx2 = tx.subscribe();
tx.send("to all".into()).unwrap();
```

**watch** — latest value only, no queue:
```rust
let (tx, mut rx) = tokio::sync::watch::channel("initial");
tx.send("updated").unwrap();
rx.changed().await.unwrap();
println!("{}", *rx.borrow()); // "updated"
```

**Actor pattern** — channels as message queues:
```rust
enum Cmd {
    Get { key: String, resp: oneshot::Sender<Option<String>> },
    Set { key: String, value: String },
}
```

## Common Pitfalls

- Using unbounded channels — no backpressure, memory grows unbounded
- Dropping all senders without the receiver noticing — check for `None`
- Using broadcast when watch suffices — broadcast buffers every message

## Best Practices

- Prefer bounded `mpsc` with reasonable buffer sizes
- Use `oneshot` for request-response in actor patterns
- Use `watch` for configuration that only needs the latest value

## Summary

`mpsc` for many-to-one with backpressure. `oneshot` for single request-response. `broadcast` for pub/sub fan-out. `watch` for latest-value state. The actor pattern combines `mpsc` + `oneshot` for structured concurrency.

## Code Examples

**Actor pattern using mpsc for commands and oneshot for responses**

```rust
use tokio::sync::{mpsc, oneshot};

enum Command {
    Get { key: String, resp: oneshot::Sender<Option<String>> },
    Set { key: String, value: String },
}

async fn actor(mut rx: mpsc::Receiver<Command>) {
    let mut state = std::collections::HashMap::new();
    while let Some(cmd) = rx.recv().await {
        match cmd {
            Command::Get { key, resp } => {
                let _ = resp.send(state.get(&key).cloned());
            }
            Command::Set { key, value } => {
                state.insert(key, value);
            }
        }
    }
}
```


## Resources

- [Tokio Channels](https://tokio.rs/tokio/tutorial/channels) — Complete guide to Tokio's async channels

---

> 📘 *This lesson is part of the [Asynchronous Rust](https://stanza.dev/courses/rust-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*