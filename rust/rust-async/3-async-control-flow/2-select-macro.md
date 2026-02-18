---
source_course: "rust-async"
source_lesson: "rust-async-select-macro"
---

# Racing Futures with select!

`select!` waits for the **first** branch to complete.

```rust
use tokio::select;

let result = select! {
    val = task_a() => {
        println!("A finished first: {val}");
        val
    }
    val = task_b() => {
        println!("B finished first: {val}");
        val
    }
};
```

## Timeout Pattern

```rust
use tokio::time::{timeout, Duration};

match timeout(Duration::from_secs(5), slow_operation()).await {
    Ok(result) => println!("Got: {result:?}"),
    Err(_) => println!("Timed out!"),
}

// Or with select!
select! {
    result = slow_operation() => println!("Result: {result:?}"),
    _ = tokio::time::sleep(Duration::from_secs(5)) => println!("Timeout!"),
}
```

## Cancellation

When one branch completes, others are **dropped** (cancelled):

```rust
select! {
    _ = operation_a() => println!("A won"),
    _ = operation_b() => println!("B won"),
}
// The losing operation is cancelled!
```

## Biased Selection

By default, `select!` randomly picks a ready branch. Use `biased;` for priority:

```rust
select! {
    biased;
    _ = high_priority() => { },  // Checked first
    _ = low_priority() => { },   // Checked second
}
```

## Loop with select!

```rust
loop {
    select! {
        Some(msg) = rx.recv() => handle(msg),
        _ = shutdown_signal.recv() => {
            println!("Shutting down");
            break;
        }
    }
}
```

See [Select](https://tokio.rs/tokio/tutorial/select).

## Code Examples

**Graceful shutdown pattern**

```rust
use tokio::{select, signal};

async fn graceful_shutdown() {
    let mut interval = tokio::time::interval(Duration::from_secs(1));
    
    loop {
        select! {
            _ = interval.tick() => {
                println!("Tick!");
                do_periodic_work().await;
            }
            _ = signal::ctrl_c() => {
                println!("Received Ctrl+C, shutting down...");
                cleanup().await;
                break;
            }
        }
    }
}
```


---

> 📘 *This lesson is part of the [Asynchronous Rust](https://stanza.dev/courses/rust-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*