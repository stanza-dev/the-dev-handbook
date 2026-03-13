---
source_course: "rust-concurrency"
source_lesson: "rust-concurrency-channel-backpressure"
---

# Backpressure & Channel Sizing

## Introduction

Choosing between bounded and unbounded channels is one of the most important design decisions in concurrent systems. An unbounded channel can silently consume all available memory if the producer outpaces the consumer, while a poorly sized bounded channel can cause unnecessary blocking. This lesson covers the mechanics of backpressure, strategies for sizing bounded channels, and patterns for monitoring channel health.

## Key Concepts

- **Backpressure**: A mechanism that slows down producers when consumers cannot keep up. Bounded channels provide natural backpressure by blocking sends when the buffer is full.
- **Buffer Sizing**: The capacity of a bounded channel determines how much temporary imbalance between producer and consumer speeds is tolerated. Too small wastes throughput on blocking; too large wastes memory.
- **Rendezvous Channel**: A bounded channel with capacity 0. Every send blocks until a receiver is ready, guaranteeing synchronous hand-off and zero buffering.
- **Channel Saturation**: When a bounded channel's buffer is consistently full, it indicates the consumer is the bottleneck. Monitoring saturation helps identify performance problems.

## Real World Context

Production systems like log aggregators, message queues, and streaming pipelines all face the producer-consumer speed mismatch problem. An unbounded log channel can cause an out-of-memory crash during a log storm. A bounded channel with backpressure instead slows down the logging threads, keeping memory usage predictable at the cost of slightly slower request handling. Understanding these trade-offs is essential for building reliable systems.

## Deep Dive

The fundamental difference between bounded and unbounded channels is what happens when the consumer is slow.

With an unbounded channel, the producer never blocks, but memory grows without limit.

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

let (tx, rx) = mpsc::channel(); // unbounded

// Fast producer
thread::spawn(move || {
    for i in 0..1_000_000 {
        tx.send(i).unwrap(); // never blocks — messages buffer in memory
    }
});

// Slow consumer
for msg in rx {
    thread::sleep(Duration::from_micros(10)); // 10x slower than producer
    // Hundreds of thousands of messages pile up in memory
}
```

In this scenario, the producer sends messages far faster than the consumer processes them. With an unbounded channel, all those messages accumulate in memory, potentially causing an OOM crash.

With a bounded channel, the producer blocks when the buffer is full, naturally throttling it to the consumer's speed.

```rust
use std::sync::mpsc;

let (tx, rx) = mpsc::sync_channel(100); // bounded: 100 slots

// When the buffer fills up, send() blocks until the consumer frees a slot.
// This keeps memory usage bounded at ~100 messages worth of data.
```

The buffer size directly controls the trade-off between throughput and memory usage. A larger buffer smooths out temporary speed mismatches but uses more memory.

A rendezvous channel (`bounded(0)` in crossbeam or `sync_channel(0)` in std) takes this to the extreme: zero buffering, pure synchronous hand-off.

```rust
use std::sync::mpsc;
use std::thread;

let (tx, rx) = mpsc::sync_channel(0); // rendezvous

thread::spawn(move || {
    tx.send(42).unwrap(); // blocks until receiver calls recv()
    println!("send completed — receiver has the value");
});

thread::sleep(std::time::Duration::from_secs(1));
let val = rx.recv().unwrap(); // unblocks the sender
println!("received: {val}");
```

The sender blocks until the receiver is ready, guaranteeing that the sender knows the message has been received before it continues. This is useful for synchronization points where you need confirmation of delivery.

To monitor channel health, you can use `try_send` to detect when the buffer is full.

```rust
use crossbeam_channel::bounded;

let (tx, rx) = bounded(1000);

// Monitor saturation
let len = tx.len().unwrap_or(0);
let capacity = 1000;
let saturation = len as f64 / capacity as f64;
if saturation > 0.8 {
    eprintln!("WARNING: channel is {:.0}% full", saturation * 100.0);
}
```

This gives you early warning that the consumer is falling behind, allowing you to scale up consumers or reduce producer throughput before the system stalls.

## Common Pitfalls

1. **Using unbounded channels in production** — Unbounded channels are convenient for prototyping but dangerous in production. A burst of messages can exhaust memory before you notice. Always use bounded channels in production systems.
2. **Sizing too small** — A buffer of 1 forces the producer to block on almost every send, serializing the pipeline and eliminating the benefit of concurrency. Size the buffer to absorb typical burst sizes.

## Best Practices

1. **Size bounded channels to 2-10x the expected burst size** — This smooths out temporary speed mismatches without wasting memory. Profile under realistic load to tune the size.
2. **Monitor channel saturation** — Track the fill level of your bounded channels. Consistently high saturation indicates the consumer is a bottleneck and needs optimization or scaling.

## Summary

- Unbounded channels never block sends but can cause OOM crashes when producers outpace consumers.
- Bounded channels provide natural backpressure by blocking sends when the buffer is full.
- Rendezvous channels (capacity 0) guarantee synchronous hand-off with zero buffering.
- Size bounded channels to absorb expected burst sizes; monitor saturation to detect bottlenecks.

## Code Examples

**Bounded channel with backpressure: the fast producer blocks when the 10-slot buffer is full**

```rust
use crossbeam_channel::bounded;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = bounded(10); // small buffer for demonstration

    // Producer: sends as fast as possible
    let producer = thread::spawn(move || {
        for i in 0..50 {
            tx.send(i).unwrap(); // blocks when buffer is full
            println!("sent: {i}");
        }
    });

    // Consumer: processes slowly
    let consumer = thread::spawn(move || {
        for msg in rx {
            thread::sleep(Duration::from_millis(50));
            println!("processed: {msg}");
        }
    });

    producer.join().unwrap();
    consumer.join().unwrap();
}
```


## Resources

- [std::sync::mpsc::sync_channel](https://doc.rust-lang.org/std/sync/mpsc/fn.sync_channel.html) — Standard library bounded channel with backpressure
- [crossbeam-channel bounded](https://docs.rs/crossbeam-channel/latest/crossbeam_channel/fn.bounded.html) — Crossbeam bounded channel with MPMC support

---

> 📘 *This lesson is part of the [Concurrent Rust](https://stanza.dev/courses/rust-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*