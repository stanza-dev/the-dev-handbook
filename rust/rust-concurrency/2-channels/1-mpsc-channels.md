---
source_course: "rust-concurrency"
source_lesson: "rust-concurrency-mpsc-channels"
---

# MPSC Channels

## Introduction

Channels are Rust's primary tool for message passing between threads. The standard library provides `std::sync::mpsc` — a Multi-Producer, Single-Consumer channel — that lets multiple threads send values to a single receiver. Channels transfer ownership of data, so there is no shared mutable state and no need for locks.

## Key Concepts

- **Sender<T> / Receiver<T>**: The two halves of a channel. `Sender` can be cloned to create multiple producers; `Receiver` cannot be cloned.
- **Unbounded Channel**: `mpsc::channel()` creates an unbounded (asynchronous) channel. Sends never block, but memory usage grows without bound if the receiver is slow.
- **Bounded Channel**: `mpsc::sync_channel(n)` creates a bounded (synchronous) channel with a fixed buffer of size `n`. Sends block when the buffer is full, providing natural backpressure.
- **Channel Closure**: When all `Sender`s are dropped, the channel closes. The `Receiver` will return `Err` (or the iterator will end), signaling that no more messages will arrive.

## Real World Context

Channels are the backbone of producer-consumer architectures. A web server might use channels to send parsed requests from acceptor threads to a pool of worker threads. A logging system might funnel log messages from many threads into a single writer thread. The ownership-transfer semantics mean you never need to worry about two threads accessing the same message simultaneously.

## Deep Dive

Creating and using a basic channel is straightforward. The `send` method moves the value into the channel, and `recv` blocks until a value arrives.

```rust
use std::sync::mpsc;
use std::thread;

let (tx, rx) = mpsc::channel();

thread::spawn(move || {
    let message = String::from("hello from the thread");
    tx.send(message).unwrap();
    // `message` has been moved — it cannot be used here
});

let received = rx.recv().unwrap();
println!("got: {received}");
```

To create multiple producers, clone the `Sender` before moving it into each thread. You must drop the original `Sender` so the channel closes when all clones are dropped.

```rust
use std::sync::mpsc;
use std::thread;

let (tx, rx) = mpsc::channel();

for i in 0..5 {
    let tx_clone = tx.clone();
    thread::spawn(move || {
        tx_clone.send(i * 10).unwrap();
    });
}

drop(tx); // drop the original so the channel closes when all clones finish

for value in rx {
    println!("received: {value}");
}
println!("channel closed, all senders dropped");
```

The `Receiver` supports several receiving modes for different use cases.

```rust
use std::sync::mpsc::{self, TryRecvError, RecvTimeoutError};
use std::time::Duration;

let (tx, rx) = mpsc::channel::<i32>();

// Blocking: waits indefinitely for a message
let msg = rx.recv().unwrap();

// Non-blocking: returns immediately
match rx.try_recv() {
    Ok(msg) => println!("got {msg}"),
    Err(TryRecvError::Empty) => println!("nothing yet"),
    Err(TryRecvError::Disconnected) => println!("channel closed"),
}

// Timed: blocks for at most the given duration
match rx.recv_timeout(Duration::from_secs(2)) {
    Ok(msg) => println!("got {msg}"),
    Err(RecvTimeoutError::Timeout) => println!("timed out"),
    Err(RecvTimeoutError::Disconnected) => println!("channel closed"),
}
```

For bounded channels, use `sync_channel`. The sender blocks when the buffer is full, which provides backpressure and prevents memory from growing without limit.

```rust
use std::sync::mpsc;

let (tx, rx) = mpsc::sync_channel(3); // buffer capacity of 3

tx.send(1).unwrap(); // buffered
tx.send(2).unwrap(); // buffered
tx.send(3).unwrap(); // buffered
// tx.send(4).unwrap(); // would BLOCK until the receiver consumes a value

println!("{}", rx.recv().unwrap()); // 1
```

## Common Pitfalls

1. **Forgetting to drop the original sender** — If you clone `tx` for each thread but never drop the original, the channel never closes and the `for value in rx` loop runs forever.
2. **Unwrapping `send` after receiver is dropped** — If the `Receiver` is dropped (or goes out of scope), `send` returns `Err`. Always handle this case in long-lived producers.

## Best Practices

1. **Use the iterator interface** — `for msg in rx { ... }` is the cleanest way to consume messages. It automatically stops when the channel closes.
2. **Choose bounded channels for production systems** — Unbounded channels can cause out-of-memory crashes if the producer outpaces the consumer. Bounded channels enforce backpressure.

## Summary

- `mpsc::channel()` creates an unbounded multi-producer, single-consumer channel.
- `mpsc::sync_channel(n)` creates a bounded channel with backpressure.
- Clone `Sender` for multiple producers; drop all senders to close the channel.
- `recv()`, `try_recv()`, and `recv_timeout()` offer blocking, non-blocking, and timed receive modes.

## Code Examples

**Stream messages through a channel and consume them with the iterator interface**

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

let (tx, rx) = mpsc::channel();

thread::spawn(move || {
    let messages = vec!["one", "two", "three"];
    for msg in messages {
        tx.send(msg).unwrap();
        thread::sleep(Duration::from_millis(200));
    }
});

// Receive as an iterator — stops when sender is dropped
for received in rx {
    println!("Got: {received}");
}
```


## Resources

- [Using Message Passing to Transfer Data Between Threads](https://doc.rust-lang.org/book/ch16-02-message-passing.html) — Official Rust Book chapter on channels and message passing
- [std::sync::mpsc module](https://doc.rust-lang.org/std/sync/mpsc/index.html) — Standard library reference for MPSC channels

---

> 📘 *This lesson is part of the [Concurrent Rust](https://stanza.dev/courses/rust-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*