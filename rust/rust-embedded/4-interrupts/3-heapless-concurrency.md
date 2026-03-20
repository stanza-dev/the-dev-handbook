---
source_course: "rust-embedded"
source_lesson: "rust-embedded-heapless-concurrency"
---

# Lock-Free Communication with heapless

## Introduction
Critical sections and RTIC locks work well for shared state, but sometimes you need a different pattern: a producer-consumer queue where an interrupt handler produces data and the main loop consumes it. The `heapless` crate provides lock-free, fixed-capacity data structures designed for exactly this use case.

## Key Concepts
- **Lock-free**: Operations complete without disabling interrupts or holding locks — they use atomic instructions instead.
- **SPSC queue**: Single-Producer, Single-Consumer queue — one writer and one reader can operate simultaneously without synchronization.
- **`heapless::spsc::Queue`**: A fixed-capacity, lock-free ring buffer for passing data from interrupt handlers to main code.

## Real World Context
A UART interrupt handler receives bytes one at a time. Rather than processing each byte in the ISR (slow) or using a critical section for every byte (high latency), you push bytes into an SPSC queue in the ISR and drain the queue in the main loop. This is the standard pattern for any streaming peripheral.

## Deep Dive

### SPSC Queue with heapless v0.8+

In `heapless` v0.8+, the queue is split into `Producer` and `Consumer` halves that can be safely held in different contexts:

```rust
use heapless::spsc::Queue;

// Create the queue with capacity 64
static QUEUE: Queue<u8, 64> = Queue::new();

fn main() {
    // Split into producer and consumer halves
    // SAFETY: split() must be called exactly once
    let (mut producer, mut consumer) = unsafe { QUEUE.split() };

    // Store producer in a global for the ISR
    // (use Mutex<RefCell<Option<Producer>>> pattern)
    // ...

    loop {
        // Drain all available items
        while let Some(byte) = consumer.dequeue() {
            process_byte(byte);
        }
        cortex_m::asm::wfi();
    }
}
```

The `split()` method returns a `Producer` and `Consumer` pair. The producer can `enqueue()` from an interrupt handler while the consumer `dequeue()`s from the main loop — simultaneously, with no locks.

### Passing the Producer to an ISR

To make the producer available in an interrupt handler, use the `Mutex<RefCell<Option<>>>` pattern:

```rust
use cortex_m::interrupt::{free, Mutex};
use core::cell::RefCell;
use heapless::spsc::{Queue, Producer};

static QUEUE: Queue<u8, 64> = Queue::new();
static TX_PRODUCER: Mutex<RefCell<Option<Producer<'static, u8, 64>>>> =
    Mutex::new(RefCell::new(None));

fn setup() {
    let (producer, consumer) = unsafe { QUEUE.split() };
    free(|cs| {
        TX_PRODUCER.borrow(cs).replace(Some(producer));
    });
    // Use consumer in main loop
}

#[interrupt]
fn USART2() {
    free(|cs| {
        if let Some(prod) = TX_PRODUCER.borrow(cs).borrow_mut().as_mut() {
            let byte = read_uart_byte();
            prod.enqueue(byte).ok(); // Drop if queue full
        }
    });
}
```

The critical section here is very short — just the enqueue operation. The actual data processing happens outside with interrupts enabled.

### Other heapless Types for Concurrency

Beyond SPSC queues, `heapless` provides additional fixed-capacity types:

```rust
use heapless::{Vec, String, LinearMap, FnvIndexMap};

// Fixed-capacity vector — no heap allocation
let mut buffer: Vec<u8, 256> = Vec::new();
buffer.extend_from_slice(b"sensor data").unwrap();

// Fixed-capacity hash map (FNV hashing)
let mut lookup: FnvIndexMap<&str, u32, 16> = FnvIndexMap::new();
lookup.insert("temperature", 2500).unwrap();
```

All these types live on the stack (or in statics) and never allocate heap memory.

## Common Pitfalls
1. **Calling `split()` more than once** — `split()` is unsafe because calling it twice creates two producers or two consumers for the same queue, breaking the SPSC guarantee. Call it exactly once.
2. **Ignoring `enqueue()` failures** — When the queue is full, `enqueue()` returns `Err`. In an ISR, you usually drop the value (`.ok()`), but you should increment an overflow counter to detect data loss.
3. **Using the wrong heapless version** — The SPSC queue API changed significantly between v0.7 and v0.8. Make sure your code matches the version in Cargo.toml.

## Best Practices
1. **Size queues for worst-case burst** — If your UART runs at 115200 baud and the main loop processes data every 10ms, the queue needs at least 115 bytes. Add margin for jitter.
2. **Use SPSC queues for ISR-to-main communication** — This is the idiomatic pattern. Avoid sharing complex state between ISRs and main code.
3. **Monitor queue fill level** — Periodically check `len()` in the main loop to detect if you are falling behind. This is an early warning for throughput problems.

## Summary
- `heapless::spsc::Queue` is a lock-free SPSC ring buffer for ISR-to-main communication.
- Call `split()` exactly once to get a `Producer` and `Consumer` pair.
- The producer can be used in an ISR, the consumer in the main loop — no locks needed during normal operation.
- Size queues for worst-case burst rates and monitor fill levels.
- All heapless types are fixed-capacity with no heap allocation.

## Code Examples

**Lock-free SPSC queue pattern — producer in ISR, consumer in main loop, with overflow tracking for data loss detection**

```rust
use heapless::spsc::Queue;
use core::sync::atomic::{AtomicU32, Ordering};

// Fixed-capacity lock-free queue: 64 bytes, no heap
static QUEUE: Queue<u8, 64> = Queue::new();
static OVERFLOW_COUNT: AtomicU32 = AtomicU32::new(0);

// Called once during initialization
fn setup() -> heapless::spsc::Consumer<'static, u8, 64> {
    // SAFETY: split() called exactly once
    let (producer, consumer) = unsafe { QUEUE.split() };
    // Store producer for ISR (via Mutex<RefCell<Option<>>>)
    store_producer_in_global(producer);
    consumer
}

// In the UART interrupt handler — enqueue received byte
fn isr_enqueue(producer: &mut heapless::spsc::Producer<'static, u8, 64>) {
    let byte = read_uart_byte();
    if producer.enqueue(byte).is_err() {
        // Queue full — track data loss
        OVERFLOW_COUNT.fetch_add(1, Ordering::Relaxed);
    }
}

// In main loop — drain and process
fn process(consumer: &mut heapless::spsc::Consumer<'static, u8, 64>) {
    while let Some(byte) = consumer.dequeue() {
        handle_received_byte(byte);
    }
}
```


## Resources

- [heapless::spsc documentation](https://docs.rs/heapless/latest/heapless/spsc/) — API reference for the lock-free SPSC queue in heapless

---

> 📘 *This lesson is part of the [Rust for Embedded Systems](https://stanza.dev/courses/rust-embedded) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*