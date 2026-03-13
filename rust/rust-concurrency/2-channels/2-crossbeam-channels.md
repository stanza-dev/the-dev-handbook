---
source_course: "rust-concurrency"
source_lesson: "rust-concurrency-crossbeam-channels"
---

# Crossbeam Channels

## Introduction

The standard library's `mpsc` channels are limited to a single consumer. The `crossbeam-channel` crate lifts this restriction, offering MPMC (Multi-Producer, Multi-Consumer) channels with superior performance, a powerful `select!` macro for multiplexing, and convenient timer channels. It has become the de facto standard for channel-based concurrency in Rust.

## Key Concepts

- **MPMC**: Both `Sender` and `Receiver` can be cloned. Multiple threads can receive from the same channel, and each message is delivered to exactly one receiver.
- **`select!` Macro**: Waits on multiple channel operations simultaneously — similar to Go's `select` statement. Supports `recv`, `send`, `default` (non-blocking fallback), and timeouts.
- **Bounded vs Unbounded**: `bounded(n)` creates a channel with a fixed-size buffer. `unbounded()` creates one that grows without limit.
- **Rendezvous Channel**: `bounded(0)` creates a zero-capacity channel where every send blocks until a receiver is ready, guaranteeing synchronous hand-off.

## Real World Context

Crossbeam channels are used extensively in production Rust systems. Work-stealing thread pools use MPMC channels as task queues. The `select!` macro enables event loops that listen for messages from multiple sources — new connections, shutdown signals, and timer ticks — all in one place. The zero-capacity rendezvous channel is useful for strict synchronization points where you need to guarantee the receiver has the message before the sender continues.

## Deep Dive

Creating crossbeam channels looks similar to `std::sync::mpsc`, but both halves are cloneable.

```rust
use crossbeam_channel::{unbounded, bounded};

// Unbounded — sends never block
let (tx, rx) = unbounded::<String>();

// Bounded — sends block when buffer is full
let (tx, rx) = bounded::<i32>(10);

// MPMC: clone the receiver for multiple consumers
let rx2 = rx.clone();
```

The `select!` macro is crossbeam's killer feature. It waits on multiple channel operations and executes whichever is ready first.

```rust
use crossbeam_channel::{select, unbounded};

let (tx1, rx1) = unbounded();
let (tx2, rx2) = unbounded();

tx1.send("from channel 1").unwrap();

select! {
    recv(rx1) -> msg => println!("rx1: {:?}", msg.unwrap()),
    recv(rx2) -> msg => println!("rx2: {:?}", msg.unwrap()),
    default => println!("neither channel had a message"),
}
```

Crossbeam provides `tick` and `after` helper functions that return channels which fire on a schedule, making timer-based logic composable with `select!`.

```rust
use crossbeam_channel::{select, tick, after};
use std::time::Duration;

let ticker = tick(Duration::from_millis(500));
let deadline = after(Duration::from_secs(3));

loop {
    select! {
        recv(ticker) -> _ => println!("tick!"),
        recv(deadline) -> _ => {
            println!("deadline reached, shutting down");
            break;
        }
    }
}
```

MPMC channels naturally enable the work-queue pattern, where multiple worker threads pull jobs from a shared channel.

```rust
use crossbeam_channel::bounded;
use std::thread;

let (tx, rx) = bounded::<usize>(64);

// Spawn 4 worker threads sharing the receiver
let workers: Vec<_> = (0..4)
    .map(|id| {
        let rx = rx.clone();
        thread::spawn(move || {
            for job in rx {
                println!("worker {id} processing job {job}");
            }
        })
    })
    .collect();

// Send 20 jobs
for job in 0..20 {
    tx.send(job).unwrap();
}
drop(tx); // close the channel so workers exit their loops

for w in workers {
    w.join().unwrap();
}
```

The key differences between bounded and unbounded channels are summarized below.

| Property | `bounded(n)` | `unbounded()` |
|---|---|---|
| `send` behavior | Blocks when buffer is full | Never blocks |
| Memory usage | Capped at `n` elements | Grows without limit |
| Backpressure | Yes | No |
| Rendezvous (`n=0`) | Sender blocks until receiver is ready | N/A |

## Common Pitfalls

1. **Unbounded channels causing OOM** — If producers are faster than consumers, an unbounded channel will grow until memory is exhausted. Use `bounded` channels in production to enforce backpressure.
2. **Forgetting `default` in `select!`** — Without a `default` arm, `select!` blocks until one operation is ready. Add `default` if you need non-blocking behavior.

## Best Practices

1. **Use crossbeam over `std::sync::mpsc` when you need multiple consumers or `select!`** — The standard library channel is fine for simple single-consumer cases, but crossbeam is more performant and more flexible.
2. **Use `bounded(0)` for rendezvous synchronization** — When you need to guarantee that the sender does not proceed until the receiver has the message, a zero-capacity channel is the cleanest solution.

## Summary

- `crossbeam-channel` provides MPMC channels where both sender and receiver can be cloned.
- The `select!` macro multiplexes over multiple channel operations, including timer channels created with `tick` and `after`.
- `bounded(n)` enforces backpressure; `bounded(0)` creates a rendezvous channel.
- MPMC channels enable clean work-queue patterns with multiple consumer threads.

## Code Examples

**MPMC work queue with bounded channel and multiple consumer threads**

```rust
use crossbeam_channel::{bounded, Receiver};
use std::thread;

fn worker(id: usize, jobs: Receiver<usize>) {
    for job in jobs {
        println!("Worker {id} processing job {job}");
        thread::sleep(std::time::Duration::from_millis(50));
    }
}

fn main() {
    let (tx, rx) = bounded(10);

    // Spawn 4 workers sharing the same receiver (MPMC)
    let handles: Vec<_> = (0..4)
        .map(|i| {
            let rx = rx.clone();
            thread::spawn(move || worker(i, rx))
        })
        .collect();

    // Send 20 jobs
    for job in 0..20 {
        tx.send(job).unwrap();
    }
    drop(tx); // close channel

    for h in handles {
        h.join().unwrap();
    }
}
```


## Resources

- [crossbeam-channel documentation](https://docs.rs/crossbeam-channel/) — API reference for crossbeam channels
- [crossbeam-channel GitHub](https://github.com/crossbeam-rs/crossbeam/tree/master/crossbeam-channel) — Source code and examples for crossbeam channels

---

> 📘 *This lesson is part of the [Concurrent Rust](https://stanza.dev/courses/rust-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*