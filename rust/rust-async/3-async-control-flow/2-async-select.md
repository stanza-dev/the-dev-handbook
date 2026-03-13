---
source_course: "rust-async"
source_lesson: "rust-async-select"
---

# Racing with select!

## Introduction

`tokio::select!` waits for the **first** future to complete and **cancels** the rest. It's essential for timeouts, graceful shutdown, and multiplexing multiple event sources.

## Key Concepts

```rust
use tokio::select;

let result = select! {
    val = task_a() => val,
    val = task_b() => val,
};
// Whichever finishes first wins; the other is dropped
```

## Real World Context

Timeout patterns, graceful shutdown with Ctrl+C, multiplexing channels and timers in event loops.

## Deep Dive

**Timeout pattern:**
```rust
use tokio::time::{timeout, Duration};

match timeout(Duration::from_secs(5), slow_op()).await {
    Ok(result) => println!("Got: {result:?}"),
    Err(_) => println!("Timed out!"),
}
```

**Cancellation:** When one branch completes, all other branches are **dropped**:
```rust
select! {
    _ = operation_a() => println!("A won"),
    _ = operation_b() => println!("B won"),
}
// The loser is cancelled immediately
```

**Biased selection** — by default `select!` randomly picks among ready branches. Use `biased;` for deterministic priority:
```rust
select! {
    biased;
    _ = high_priority() => { }  // Checked first
    _ = low_priority() => { }   // Checked second
}
```

**Loop + select** for event loops:
```rust
loop {
    select! {
        Some(msg) = rx.recv() => handle(msg),
        _ = shutdown.recv() => break,
    }
}
```

## Common Pitfalls

- Not realizing losing branches are cancelled/dropped
- Using `select!` with non-cancellation-safe operations
- Forgetting `biased;` when priority matters

## Best Practices

- Pair `select!` with cancellation-safe operations (next section)
- Use `biased;` for shutdown signals to ensure they're checked first
- Prefer `tokio::time::timeout` over manual select+sleep for simple timeouts

## Summary

`select!` races futures — first to finish wins, losers are dropped. Use `biased;` for priority. The timeout pattern is the most common use case. Always consider cancellation safety.

## Code Examples

**Graceful shutdown event loop with biased select**

```rust
use tokio::{select, signal, time::{interval, Duration}};

async fn event_loop() {
    let mut tick = interval(Duration::from_secs(1));

    loop {
        select! {
            biased;
            _ = signal::ctrl_c() => {
                println!("Shutting down...");
                break;
            }
            _ = tick.tick() => {
                println!("Tick!");
            }
        }
    }
}
```


## Resources

- [Tokio Select](https://tokio.rs/tokio/tutorial/select) — Tokio tutorial on the select! macro

---

> 📘 *This lesson is part of the [Asynchronous Rust](https://stanza.dev/courses/rust-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*