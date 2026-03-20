---
source_course: "rust-embedded"
source_lesson: "rust-embedded-hal-traits"
---

# embedded-hal: Portable Hardware Drivers

## Introduction
The `embedded-hal` crate defines a set of traits — `OutputPin`, `InputPin`, `I2c`, `SpiDevice`, `DelayNs` — that abstract over hardware peripherals. A driver written against these traits works on any microcontroller that implements them, from STM32 to ESP32 to nRF52.

## Key Concepts
- **`embedded-hal` traits**: Platform-agnostic interfaces for GPIO, I2C, SPI, UART, timers, and ADC.
- **HAL crate**: A platform-specific implementation of `embedded-hal` traits (e.g., `stm32f4xx-hal`, `esp-hal`).
- **Portable driver**: A device driver generic over `embedded-hal` traits, usable on any platform.

## Real World Context
Imagine writing a driver for a BME280 temperature sensor. Without `embedded-hal`, you would write one driver for STM32, another for nRF52, another for ESP32 — all doing the same I2C operations. With `embedded-hal`, you write the driver once and it works everywhere.

## Deep Dive

### Digital I/O

The most basic traits control GPIO pins. `OutputPin` lets you drive a pin high or low:

```rust
use embedded_hal::digital::OutputPin;

fn blink<P: OutputPin>(pin: &mut P) {
    pin.set_high().unwrap();
    // ... delay ...
    pin.set_low().unwrap();
}
```

This function works with any pin on any microcontroller — the caller provides the concrete type.

### Delays

The `DelayNs` trait provides nanosecond, microsecond, and millisecond delays:

```rust
use embedded_hal::delay::DelayNs;

fn wait_then_read<D: DelayNs>(delay: &mut D) {
    delay.delay_ms(100);
}
```

The concrete implementation might use a hardware timer, a SysTick counter, or a busy loop — the trait hides that detail.

### I2C

The `I2c` trait covers read, write, and write-then-read transactions:

```rust
use embedded_hal::i2c::I2c;

fn read_sensor<I: I2c>(i2c: &mut I, addr: u8) -> Result<u16, I::Error> {
    let mut buf = [0u8; 2];
    i2c.write_read(addr, &[0x00], &mut buf)?;
    Ok(u16::from_be_bytes(buf))
}
```

The `write_read` method sends a register address and reads back data in a single transaction. The `I::Error` associated type lets each platform report its own error details.

### SPI

The `SpiDevice` trait manages chip-select and full-duplex transfers:

```rust
use embedded_hal::spi::SpiDevice;

fn transfer<S: SpiDevice>(spi: &mut S, data: &mut [u8]) -> Result<(), S::Error> {
    spi.transfer_in_place(data)?;
    Ok(())
}
```

The `SpiDevice` trait automatically asserts/deasserts the chip-select line around the transfer.

### The Ecosystem Stack

The embedded Rust ecosystem forms a layered stack. Your application code sits on top, using portable drivers, which depend on `embedded-hal` traits, implemented by platform HAL crates, built on auto-generated PACs:

```
+------------------+
|   Your Code      |
+------------------+
|  Device Drivers  |  (e.g., bme280, ssd1306)
+------------------+
|  embedded-hal    |  (traits)
+------------------+
|   HAL Crate      |  (e.g., stm32f4xx-hal)
+------------------+
|      PAC         |  (e.g., stm32f4)
+------------------+
|    Hardware      |
+------------------+
```

This layering means switching from an STM32 to an nRF52 only requires changing the HAL crate — your drivers and application logic stay the same.

## Common Pitfalls
1. **Mixing embedded-hal versions** — The ecosystem has v0.2 and v1.0 drivers. Make sure your HAL crate and device drivers target the same embedded-hal version, or use the `embedded-hal-compat` shim.
2. **Ignoring error types** — Calling `.unwrap()` on I2C/SPI errors causes a panic on bus faults. In production, handle errors gracefully with retry logic or fallback behavior.
3. **Assuming blocking is OK** — All traits above are blocking. For async, use `embedded-hal-async`.

## Best Practices
1. **Write drivers as libraries generic over traits** — Accept `impl I2c` or `impl SpiDevice` so users can plug in any HAL implementation.
2. **Use `embedded-hal` v1.0+** — The 1.0 release is stable and the ecosystem is converging on it. Avoid v0.2 for new projects.
3. **Test drivers with `embedded-hal-mock`** — The mock crate provides fake implementations of all traits so you can unit-test your driver on the host.

## Summary
- `embedded-hal` defines standard traits for GPIO, I2C, SPI, delays, and more.
- Drivers written against these traits are portable across all platforms.
- The ecosystem stack is: Hardware -> PAC -> HAL -> embedded-hal -> Drivers -> Application.
- Use v1.0+ and test with `embedded-hal-mock`.

## Code Examples

**A portable I2C driver generic over embedded-hal traits — the same code compiles for any microcontroller with an I2c implementation**

```rust
// A portable I2C temperature sensor driver
// Works on STM32, nRF52, ESP32 — any platform with an I2c impl
use embedded_hal::i2c::I2c;

pub struct TemperatureSensor<I> {
    i2c: I,
    address: u8,
}

impl<I: I2c> TemperatureSensor<I> {
    pub fn new(i2c: I, address: u8) -> Self {
        Self { i2c, address }
    }

    pub fn read_celsius(&mut self) -> Result<f32, I::Error> {
        let mut buf = [0u8; 2];
        // Send register address 0x00, read 2 bytes back
        self.i2c.write_read(self.address, &[0x00], &mut buf)?;
        // Convert raw big-endian value to Celsius
        let raw = i16::from_be_bytes(buf);
        Ok(raw as f32 / 128.0)
    }
}

// Usage on STM32:
// let i2c = stm32f4xx_hal::i2c::I2c::new(dp.I2C1, ...);
// let mut sensor = TemperatureSensor::new(i2c, 0x48);
// let temp = sensor.read_celsius().unwrap();
```


## Resources

- [embedded-hal crate documentation](https://docs.rs/embedded-hal/1.0.0/embedded_hal/) — API reference for embedded-hal 1.0 traits
- [Awesome Embedded Rust](https://github.com/rust-embedded/awesome-embedded-rust) — Curated list of embedded-hal drivers, HAL crates, and tools

---

> 📘 *This lesson is part of the [Rust for Embedded Systems](https://stanza.dev/courses/rust-embedded) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*