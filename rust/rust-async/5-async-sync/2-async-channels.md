---
source_course: "rust-async"
source_lesson: "rust-async-channels"
---

# Tokio Channels

## mpsc: Multiple Producer, Single Consumer

```rust
use tokio::sync::mpsc;

let (tx, mut rx) = mpsc::channel::<String>(100); // Buffer size

// Sender (can be cloned)
tokio::spawn(async move {
    tx.send("hello".to_string()).await.unwrap();
});

// Receiver
while let Some(msg) = rx.recv().await {
    println!("Got: {msg}");
}
```

## oneshot: Single Value

```rust
use tokio::sync::oneshot;

let (tx, rx) = oneshot::channel();

tokio::spawn(async move {
    let result = expensive_computation().await;
    tx.send(result).unwrap();
});

let value = rx.await.unwrap();
```

## broadcast: Multiple Consumers

```rust
use tokio::sync::broadcast;

let (tx, _) = broadcast::channel::<String>(16);
let mut rx1 = tx.subscribe();
let mut rx2 = tx.subscribe();

tx.send("to all".to_string()).unwrap();
// Both rx1 and rx2 receive "to all"
```

## watch: Latest Value Only

```rust
use tokio::sync::watch;

let (tx, mut rx) = watch::channel("initial");

// Sender updates value
tx.send("updated").unwrap();

// Receiver gets latest value
println!("Current: {}", *rx.borrow());

// Wait for changes
rx.changed().await.unwrap();
println!("New: {}", *rx.borrow());
```

## Channel Selection

| Type | Producers | Consumers | Buffering | Use Case |
|------|-----------|-----------|-----------|----------|
| mpsc | Many | One | Yes | Task communication |
| oneshot | One | One | No | Request-response |
| broadcast | One | Many | Yes | Pub/sub |
| watch | One | Many | Latest only | Config updates |

See [Channels](https://tokio.rs/tokio/tutorial/channels).

## Code Examples

**Actor pattern with request-response**

```rust
// Actor pattern with channels
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

- [Tokio Channels](https://tokio.rs/tokio/tutorial/channels) — Complete guide to Tokio channels

---

> 📘 *This lesson is part of the [Asynchronous Rust](https://stanza.dev/courses/rust-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*