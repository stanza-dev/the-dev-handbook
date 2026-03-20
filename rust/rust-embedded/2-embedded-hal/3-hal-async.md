---
source_course: "rust-embedded"
source_lesson: "rust-embedded-hal-async"
---

# embedded-hal-async: Non-Blocking Drivers

## Introduction
Blocking I/O wastes CPU cycles — while waiting for an I2C transfer to complete, the processor could be doing useful work. The `embedded-hal-async` crate provides async versions of the standard traits, enabling cooperative multitasking on embedded systems with executors like Embassy.

## Key Concepts
- **`embedded-hal-async`**: Async counterparts to embedded-hal traits (`I2c`, `SpiDevice`, `DelayNs`).
- **Embassy**: A Rust async runtime for embedded systems that uses hardware timers for task scheduling.
- **Cooperative multitasking**: Tasks voluntarily yield at `.await` points, allowing other tasks to run without preemption.

## Real World Context
Consider a sensor hub that reads temperature every second, updates a display every 500ms, and checks for BLE commands continuously. With blocking I/O, these tasks would require interrupts and complex state machines. With async, you write each task as a simple loop with `.await` calls.

## Deep Dive

### Async I2C

The async I2C trait mirrors the blocking one but returns futures:

```rust
use embedded_hal_async::i2c::I2c;

async fn read_sensor<I: I2c>(i2c: &mut I, addr: u8) -> Result<u16, I::Error> {
    let mut buf = [0u8; 2];
    i2c.write_read(addr, &[0x00], &mut buf).await?;
    Ok(u16::from_be_bytes(buf))
}
```

The `.await` yields control to the executor while the DMA transfer completes, instead of busy-waiting.

### Embassy Example

Embassy provides an executor, HAL integration, and timer-based delays:

```rust
#![no_std]
#![no_main]

use embassy_executor::Spawner;
use embassy_stm32::gpio::{Level, Output, Speed};
use embassy_time::Timer;

#[embassy_executor::main]
async fn main(_spawner: Spawner) {
    let p = embassy_stm32::init(Default::default());
    let mut led = Output::new(p.PA5, Level::Low, Speed::Low);

    loop {
        led.set_high();
        Timer::after_millis(500).await;
        led.set_low();
        Timer::after_millis(500).await;
    }
}
```

The `Timer::after_millis(500).await` puts the task to sleep using a hardware timer — the CPU can execute other tasks or enter a low-power sleep mode.

### Spawning Multiple Tasks

Async shines when running concurrent operations:

```rust
#[embassy_executor::task]
async fn sensor_task(mut i2c: I2cDevice) {
    loop {
        let temp = read_temperature(&mut i2c).await;
        // Process temperature
        Timer::after_secs(1).await;
    }
}

#[embassy_executor::task]
async fn display_task(mut spi: SpiDevice) {
    loop {
        update_display(&mut spi).await;
        Timer::after_millis(500).await;
    }
}
```

Both tasks run concurrently on a single core without threads, RTOS, or manual state machines.

## Common Pitfalls
1. **Blocking in an async context** — Calling a blocking function (e.g., busy-wait delay) inside an async task starves all other tasks. Always use async-aware delays and I/O.
2. **Stack overflow with too many tasks** — Each Embassy task needs its own stack. On memory-constrained MCUs, limit the number of concurrent tasks.
3. **Task memory sizing on stable** — Embassy works on stable Rust (since 1.75), but on stable it uses an arena-based allocator for tasks that requires configuring arena sizes. On nightly with `type_alias_impl_trait`, task sizes are computed at compile time automatically.

## Best Practices
1. **Use Embassy for new projects** — Embassy is the most mature async embedded runtime and supports STM32, nRF, RP2040, and ESP32.
2. **Write drivers against `embedded-hal-async` traits** — This makes them usable in both async and blocking contexts (with adapters).
3. **Measure power consumption** — Async executors can automatically enter low-power modes when all tasks are waiting, significantly reducing power draw.

## Summary
- `embedded-hal-async` provides async versions of I2C, SPI, GPIO, and delay traits.
- Embassy is the leading async runtime for embedded Rust.
- Async enables concurrent tasks on a single core without an RTOS.
- Tasks yield at `.await` points, allowing the CPU to sleep or run other tasks.
- Always use async-aware I/O to avoid blocking the executor.

## Code Examples

**Embassy application with two concurrent async tasks — LED blinking and sensor reading run cooperatively on a single core**

```rust
#![no_std]
#![no_main]

use embassy_executor::Spawner;
use embassy_stm32::gpio::{Level, Output, Speed};
use embassy_stm32::i2c::I2c;
use embassy_time::Timer;
use embedded_hal_async::i2c::I2c as I2cTrait;

// Task 1: Blink LED every 500ms
#[embassy_executor::task]
async fn blink_task(mut led: Output<'static>) {
    loop {
        led.toggle();
        Timer::after_millis(500).await; // Yields to executor
    }
}

// Task 2: Read sensor every second
#[embassy_executor::task]
async fn sensor_task(mut i2c: I2c<'static>) {
    let mut buf = [0u8; 2];
    loop {
        i2c.write_read(0x48, &[0x00], &mut buf).await.ok();
        let raw = i16::from_be_bytes(buf);
        let celsius = raw as f32 / 128.0;
        // Process reading...
        Timer::after_secs(1).await;
    }
}

#[embassy_executor::main]
async fn main(spawner: Spawner) {
    let p = embassy_stm32::init(Default::default());
    let led = Output::new(p.PA5, Level::Low, Speed::Low);
    let i2c = I2c::new_blocking(p.I2C1, p.PB8, p.PB9, Default::default());

    // Spawn concurrent tasks on a single core
    spawner.spawn(blink_task(led)).unwrap();
    spawner.spawn(sensor_task(i2c)).unwrap();
}
```


## Resources

- [Embassy project](https://embassy.dev/) — Async runtime and HAL for embedded Rust

---

> 📘 *This lesson is part of the [Rust for Embedded Systems](https://stanza.dev/courses/rust-embedded) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*