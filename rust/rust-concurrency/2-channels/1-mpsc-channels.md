---
source_course: "rust-concurrency"
source_lesson: "rust-concurrency-mpsc-channels"
---

# Multi-Producer, Single-Consumer Channels

Channels let threads communicate without shared memory.

```rust
use std::sync::mpsc;
use std::thread;

let (tx, rx) = mpsc::channel();

thread::spawn(move || {
    let val = String::from("hello");
    tx.send(val).unwrap();
    // val is moved into the channel!
});

let received = rx.recv().unwrap();
println!("Got: {received}");
```

## Multiple Producers

Clone the transmitter:

```rust
let (tx, rx) = mpsc::channel();

for i in 0..5 {
    let tx_clone = tx.clone();
    thread::spawn(move || {
        tx_clone.send(i).unwrap();
    });
}

drop(tx); // Drop original so channel closes when clones finish

for received in rx {
    println!("Got: {received}");
}
```

## Receiving Methods

```rust
// Blocking receive
let msg = rx.recv().unwrap();

// Non-blocking (returns immediately)
match rx.try_recv() {
    Ok(msg) => println!("{msg}"),
    Err(TryRecvError::Empty) => println!("No message yet"),
    Err(TryRecvError::Disconnected) => println!("Channel closed"),
}

// With timeout
match rx.recv_timeout(Duration::from_secs(1)) {
    Ok(msg) => println!("{msg}"),
    Err(RecvTimeoutError::Timeout) => println!("Timeout!"),
    Err(RecvTimeoutError::Disconnected) => println!("Closed"),
}
```

## Synchronous Channels

```rust
// Bounded channel - sender blocks when full
let (tx, rx) = mpsc::sync_channel(3); // Buffer size 3

tx.send(1).unwrap(); // OK
tx.send(2).unwrap(); // OK
tx.send(3).unwrap(); // OK
// tx.send(4).unwrap(); // Would BLOCK until receiver consumes
```

See [Message Passing](https://doc.rust-lang.org/book/ch16-02-message-passing.html).

## Code Examples

**Streaming messages**

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

// Stream of values
let (tx, rx) = mpsc::channel();

thread::spawn(move || {
    let messages = vec!["one", "two", "three"];
    for msg in messages {
        tx.send(msg).unwrap();
        thread::sleep(Duration::from_millis(200));
    }
});

// Receive as iterator
for received in rx {
    println!("Got: {received}");
}
```


## Resources

- [Message Passing](https://doc.rust-lang.org/book/ch16-02-message-passing.html) — Official guide to channels

---

> 📘 *This lesson is part of the [Concurrent Rust](https://stanza.dev/courses/rust-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*