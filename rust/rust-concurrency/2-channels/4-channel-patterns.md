---
source_course: "rust-concurrency"
source_lesson: "rust-concurrency-channel-patterns"
---

# Channel Patterns

## Introduction

Channels are more than simple pipes. By combining send/receive semantics with ownership rules and channel lifecycle, you can build powerful concurrent architectures: one-shot notifications, fan-out/fan-in work distribution, multi-stage pipelines, and OS-level pipes for process communication. This lesson covers the most important patterns.

## Key Concepts

- **Oneshot**: A channel used for exactly one message, then dropped. The sender fires a single value and the receiver waits for it — useful for returning a result from a spawned task.
- **Fan-out / Fan-in**: Fan-out distributes work from one producer to many consumers. Fan-in collects results from many producers into one consumer.
- **Pipeline**: Chains of channels where the output of one stage feeds into the input of the next, enabling staged data processing with natural buffering between stages.
- **OS Pipe**: `std::io::pipe()` (stabilized in Rust 1.87) creates an operating-system-level pipe with a `PipeReader` and `PipeWriter` that implement `Read` and `Write`, useful for inter-process communication and connecting to child process stdio.

## Real World Context

Oneshot channels are the standard way for async runtimes (like Tokio) to return results from spawned tasks. Fan-out/fan-in powers map-reduce style computations — distribute chunks to workers, collect partial results, merge. Pipeline patterns are used in data processing systems like ETL engines: each stage parses, transforms, or writes data independently, connected by bounded channels that provide backpressure. OS pipes bridge Rust threads with external processes.

## Deep Dive

### Oneshot Pattern

The simplest pattern: send one value, then let the channel drop. This is ideal for getting a single result back from a spawned thread or task.

```rust
use std::sync::mpsc;
use std::thread;

fn compute_async(input: u64) -> mpsc::Receiver<u64> {
    let (tx, rx) = mpsc::channel();
    thread::spawn(move || {
        let result = expensive_computation(input);
        let _ = tx.send(result); // send once, then tx is dropped
    });
    rx
}

// Caller gets a "future-like" receiver
let result_rx = compute_async(42);
// ... do other work ...
let result = result_rx.recv().unwrap(); // block when we need the result
```

This pattern gives the caller a "future-like" handle: they can continue doing work and only block when they actually need the computed result. The channel's lifetime naturally manages the relationship between producer and consumer.

The sender is dropped after the single send, which closes the channel. The receiver knows exactly one message will arrive.

### Fan-out / Fan-in

Fan-out distributes work items from a single source to a pool of workers. Fan-in collects their results into a single stream. With crossbeam MPMC channels, this pattern is particularly clean.

```rust
use crossbeam_channel::{bounded, Sender, Receiver};
use std::thread;

fn fan_out_fan_in(items: Vec<u64>, num_workers: usize) -> Vec<u64> {
    let (work_tx, work_rx) = bounded::<u64>(64);
    let (result_tx, result_rx) = bounded::<u64>(64);

    // Fan-out: spawn workers that pull from the shared work channel
    let workers: Vec<_> = (0..num_workers)
        .map(|_| {
            let work_rx = work_rx.clone();
            let result_tx = result_tx.clone();
            thread::spawn(move || {
                for item in work_rx {
                    result_tx.send(process(item)).unwrap();
                }
            })
        })
        .collect();
    drop(result_tx); // drop original so channel closes when workers finish

    // Send work items
    for item in items {
        work_tx.send(item).unwrap();
    }
    drop(work_tx); // close work channel — workers will exit their loops

    // Fan-in: collect all results
    let results: Vec<u64> = result_rx.iter().collect();

    for w in workers {
        w.join().unwrap();
    }

    results
}
```

The key insight is the double `drop`: dropping `result_tx` (the original) ensures the result channel closes when all worker clones finish, and dropping `work_tx` signals workers to exit their receive loops.

### Pipeline Pattern

A pipeline chains stages together. Each stage reads from an input channel, processes data, and writes to an output channel. Bounded channels between stages provide natural backpressure.

```rust
use std::sync::mpsc;
use std::thread;

// Stage 1: Parse raw data
fn parser(input: mpsc::Receiver<String>, output: mpsc::Sender<ParsedRecord>) {
    for raw in input {
        if let Ok(record) = parse(raw) {
            let _ = output.send(record);
        }
    }
}

// Stage 2: Transform records
fn transformer(input: mpsc::Receiver<ParsedRecord>, output: mpsc::Sender<TransformedRecord>) {
    for record in input {
        let _ = output.send(transform(record));
    }
}

// Stage 3: Write results
fn writer(input: mpsc::Receiver<TransformedRecord>) {
    for record in input {
        write_to_database(record);
    }
}

// Wire stages together
let (tx1, rx1) = mpsc::channel();
let (tx2, rx2) = mpsc::channel();
let (tx3, rx3) = mpsc::channel();

thread::spawn(move || parser(rx1, tx2));
thread::spawn(move || transformer(rx2, tx3));
thread::spawn(move || writer(rx3));

// Feed data into the pipeline
for line in read_lines("data.csv") {
    tx1.send(line).unwrap();
}
```

Each stage runs independently on its own thread, processing data as fast as it can. The channels between stages act as buffers, decoupling the stages so a fast parser does not overwhelm a slow writer.

### OS-Level Pipes (Rust 1.87+)

`std::io::pipe()` creates an operating-system pipe. Unlike channels, pipes work with raw bytes and implement `Read`/`Write`, making them suitable for connecting to child processes or bridging with I/O-oriented APIs.

```rust
use std::io::{self, Read, Write};
use std::thread;

let (mut reader, mut writer) = io::pipe().unwrap();

thread::spawn(move || {
    writer.write_all(b"hello from a pipe!").unwrap();
    // writer is dropped here, closing the write end
});

let mut buf = String::new();
reader.read_to_string(&mut buf).unwrap();
println!("{buf}"); // "hello from a pipe!"
```

Unlike channels, OS pipes work with raw bytes and implement standard `Read`/`Write` traits. This makes them ideal for bridging with child processes or I/O-oriented APIs that expect byte streams rather than typed messages.

## Common Pitfalls

1. **Deadlocking a pipeline** — If a downstream stage panics or hangs, upstream stages block on send. Use bounded channels with timeouts or monitor thread health to avoid silent deadlocks.
2. **Forgetting to drop senders** — In fan-out/fan-in, every clone of the result sender must be dropped for the fan-in loop to terminate. Drop the original after cloning for workers, and let worker threads drop their clones when they exit.

## Best Practices

1. **Use bounded channels between pipeline stages** — Unbounded channels let a fast producer overwhelm a slow consumer. Bounded channels enforce backpressure and keep memory usage predictable.
2. **Prefer oneshot over shared state for single results** — When a spawned thread needs to return one value, a channel is simpler and safer than an `Arc<Mutex<Option<T>>>`.

## Summary

- Oneshot: send one value, drop the channel — a clean way to return results from spawned work.
- Fan-out/fan-in: distribute work to N workers via MPMC, collect results through a shared result channel.
- Pipeline: chain stages with channels for staged data processing with natural backpressure.
- `std::io::pipe()` (Rust 1.87+) provides OS-level byte pipes that implement `Read`/`Write`.

## Code Examples

**Oneshot pattern: fire-and-forget a computation, retrieve the result later via the receiver**

```rust
use std::sync::mpsc;
use std::thread;

// Oneshot pattern: spawn work and get a receiver "handle"
fn compute_in_background(n: u64) -> mpsc::Receiver<u64> {
    let (tx, rx) = mpsc::channel();
    thread::spawn(move || {
        // Simulate expensive work
        let result = (1..=n).product();
        let _ = tx.send(result);
    });
    rx
}

fn main() {
    let factorial_rx = compute_in_background(20);

    // Do other work while computation runs...
    println!("computing in background...");

    // Block only when we need the result
    let result = factorial_rx.recv().unwrap();
    println!("20! = {result}");
}
```


## Resources

- [std::io::pipe (Rust 1.87+)](https://doc.rust-lang.org/std/io/fn.pipe.html) — OS-level pipe API stabilized in Rust 1.87
- [crossbeam-channel documentation](https://docs.rs/crossbeam-channel/) — Crossbeam channels used for fan-out/fan-in and pipeline patterns

---

> 📘 *This lesson is part of the [Concurrent Rust](https://stanza.dev/courses/rust-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*