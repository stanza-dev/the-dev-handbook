---
source_course: "rust-embedded"
source_lesson: "rust-embedded-dma-basics"
---

# DMA: Direct Memory Access

## Introduction
Polling and interrupt-driven I/O both involve the CPU for every byte transferred. DMA (Direct Memory Access) offloads bulk data transfers to a dedicated hardware controller, freeing the CPU to do other work — or sleep. DMA is essential for high-throughput applications like audio streaming, display updates, and network packet processing.

## Key Concepts
- **DMA controller**: A hardware unit that moves data between memory and peripherals (or memory-to-memory) without CPU involvement.
- **DMA channel/stream**: A configurable path in the DMA controller, bound to a peripheral and a memory buffer.
- **Transfer complete interrupt**: Fired when a DMA transfer finishes, signaling the CPU to process the received data.

## Real World Context
Reading 1024 bytes from an SPI flash chip byte-by-byte with interrupts generates 1024 interrupt entries. With DMA, you set up one transfer and get a single interrupt when all 1024 bytes are in RAM. This is the difference between 90% CPU load and 2% CPU load.

## Deep Dive

### DMA with HAL Crates

Most HAL crates provide safe DMA APIs. Here is a typical pattern with `stm32f4xx-hal`:

```rust
use stm32f4xx_hal::{
    dma::{StreamsTuple, config::DmaConfig, Transfer, MemoryToPeripheral},
    pac,
    prelude::*,
};

let dp = pac::Peripherals::take().unwrap();
let dma1 = StreamsTuple::new(dp.DMA1);

// Configure a DMA stream for USART2 TX
let config = DmaConfig::default()
    .transfer_complete_interrupt(true)
    .memory_increment(true);

let tx_buffer: &'static [u8] = b"Hello DMA!\n";
let transfer = Transfer::init_memory_to_peripheral(
    dma1.6,  // DMA1 Stream 6 (USART2_TX on STM32F4)
    usart2_tx,
    tx_buffer,
    None,
    config,
);

// Start the transfer — CPU is free to do other work
transfer.start(|_| {});
```

This sets up a memory-to-peripheral transfer: the DMA controller reads bytes from the buffer and feeds them to the USART, one at a time, without CPU intervention.

### Buffer Ownership and Safety

DMA requires the buffer to remain valid for the entire transfer duration. Rust's ownership system helps here:

```rust
// The buffer must be 'static or owned — DMA reads it asynchronously
static TX_BUF: [u8; 64] = [0; 64];

// Or use a owned buffer that the DMA transfer takes ownership of
let buffer = cortex_m::singleton!(: [u8; 256] = [0; 256]).unwrap();
```

The `singleton!` macro creates a `&'static mut` reference, ensuring the buffer lives for the entire program and cannot be accidentally freed while DMA is using it.

### Double Buffering

For continuous data streams, use two buffers: one being filled by DMA while the CPU processes the other:

```rust
// Buffer A: DMA fills this
// Buffer B: CPU processes this
// When DMA finishes A, swap: DMA fills B, CPU processes A
```

This is the standard pattern for audio codecs, ADC sampling, and display frame buffers.

## Common Pitfalls
1. **Buffer lifetime violations** — If the buffer is dropped or moved while DMA is active, the DMA controller reads/writes garbage memory. Always use `'static` buffers or ownership-transfer APIs.
2. **Cache coherency on Cortex-M7** — Cortex-M7 has a data cache. DMA writes may not be visible to the CPU without cache invalidation. Use `SCB::invalidate_dcache_by_address()` after DMA receive.
3. **Forgetting to clear the transfer complete flag** — The DMA interrupt fires once. If you do not clear the flag, it fires continuously.

## Best Practices
1. **Use HAL DMA APIs** — Manual DMA setup requires configuring stream/channel mappings, priority, FIFO thresholds, and burst sizes. HAL crates handle this.
2. **Prefer `singleton!` for DMA buffers** — It guarantees a `'static` lifetime and single initialization, preventing accidental reuse.
3. **Use double buffering for streaming** — Any application that continuously receives or transmits data should use double-buffered DMA to avoid gaps.

## Summary
- DMA moves data between peripherals and memory without CPU involvement.
- HAL crates provide safe DMA transfer APIs with ownership semantics.
- Buffers must have `'static` lifetime — use `singleton!` or `static` arrays.
- Double buffering enables continuous streaming without data loss.
- On Cortex-M7, invalidate the data cache after DMA receive transfers.

## Code Examples

**DMA receive setup using stm32f4xx-hal — the singleton! macro creates a static buffer that the DMA controller fills without CPU intervention**

```rust
use stm32f4xx_hal::{
    dma::{config::DmaConfig, Transfer, PeripheralToMemory, StreamsTuple},
    pac,
    prelude::*,
    serial::Serial,
};
use cortex_m::singleton;

fn setup_dma_receive(dp: pac::Peripherals) {
    let dma1 = StreamsTuple::new(dp.DMA1);

    // Allocate a static buffer for DMA to write into
    let rx_buf = singleton!(: [u8; 128] = [0; 128]).unwrap();

    let config = DmaConfig::default()
        .transfer_complete_interrupt(true)
        .memory_increment(true);

    // DMA1 Stream 5 is mapped to USART2_RX on STM32F4
    let mut transfer = Transfer::init_peripheral_to_memory(
        dma1.5,
        usart2_rx,
        rx_buf,
        None,
        config,
    );

    // Start receiving — CPU is free until transfer complete interrupt
    transfer.start(|_| {});
    // When interrupt fires, rx_buf contains received data
}
```


## Resources

- [STM32F4xx-hal DMA documentation](https://docs.rs/stm32f4xx-hal/latest/stm32f4xx_hal/dma/) — DMA module documentation for the STM32F4 HAL crate

---

> 📘 *This lesson is part of the [Rust for Embedded Systems](https://stanza.dev/courses/rust-embedded) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*