---
source_course: "rust-embedded"
source_lesson: "rust-embedded-testing-qemu"
---

# Testing Embedded Rust Without Hardware

## Introduction
You do not always have the target hardware on your desk — and even when you do, hardware-in-the-loop testing is slow. QEMU can emulate ARM Cortex-M targets, allowing you to run and test firmware on your development machine. Combined with host-side unit testing for pure logic, you can achieve good coverage without touching real hardware.

## Key Concepts
- **QEMU**: An open-source machine emulator that can simulate ARM Cortex-M boards.
- **Semihosting**: A mechanism that redirects I/O calls from the emulated firmware to the host terminal (e.g., printing to stdout).
- **Host-side testing**: Running `cargo test` for platform-independent logic without cross-compilation.

## Real World Context
In CI/CD pipelines for embedded projects, you cannot plug in a microcontroller. QEMU lets you run integration tests against your firmware in GitHub Actions or any CI environment. Host-side unit tests cover your parsing logic, state machines, and algorithms.

## Deep Dive

### Running Firmware in QEMU

QEMU supports several ARM boards. The LM3S6965EVB is commonly used for testing:

```bash
# Install QEMU
brew install qemu          # macOS
apt install qemu-system-arm  # Ubuntu

# Run firmware in emulator
qemu-system-arm \
    -cpu cortex-m3 \
    -machine lm3s6965evb \
    -nographic \
    -semihosting-config enable=on,target=native \
    -kernel target/thumbv7m-none-eabi/release/my-app
```

The `-semihosting-config` flag enables semihosting so your firmware can print to the host terminal.

### Semihosting Output

The `cortex-m-semihosting` crate provides `hprintln!` for debug output:

```rust
use cortex_m_semihosting::hprintln;

#[entry]
fn main() -> ! {
    hprintln!("Hello from QEMU!");
    // Continue with tests...
    loop {}
}
```

This macro formats and prints to the host terminal through the QEMU semihosting bridge. Do not use semihosting in production — it requires a debugger or emulator.

### Host-Side Unit Testing

The most effective testing strategy separates hardware-dependent code from pure logic. Test the pure logic on the host:

```rust
#![cfg_attr(not(test), no_std)]
#![cfg_attr(not(test), no_main)]

// Pure logic — testable on any platform
pub fn convert_temperature(raw: u16) -> f32 {
    (raw as f32 / 256.0) - 40.0
}

pub fn parse_packet(data: &[u8]) -> Option<SensorReading> {
    if data.len() < 4 {
        return None;
    }
    Some(SensorReading {
        temp: u16::from_be_bytes([data[0], data[1]]),
        humidity: u16::from_be_bytes([data[2], data[3]]),
    })
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_temperature_conversion() {
        assert_eq!(convert_temperature(10240), 0.0); // 40*256 raw = 0 C
    }

    #[test]
    fn test_short_packet_rejected() {
        assert!(parse_packet(&[0x00, 0x01]).is_none());
    }
}
```

The `cfg_attr` lines make the crate use `no_std` and `no_main` only when not running tests. When you run `cargo test`, it compiles for the host with full `std` access.

### cargo-embed and probe-rs

For testing on real hardware:

```bash
cargo install probe-rs-tools

# Flash and run with RTT (Real-Time Transfer) output
cargo run --release
```

RTT is faster than semihosting and works on real hardware without halting the CPU.

## Common Pitfalls
1. **Testing hardware-dependent code on the host** — Code that touches registers or peripherals cannot run in `cargo test`. Keep it in separate modules gated with `#[cfg(not(test))]`.
2. **QEMU does not emulate all peripherals** — QEMU supports basic CPU, memory, and some peripherals. Complex peripherals like DMA, ADC, or specific timer modes may not work.
3. **Semihosting in production** — Semihosting halts the CPU on each call. If no debugger is attached, the firmware hangs. Gate semihosting calls behind `#[cfg(debug_assertions)]`.

## Best Practices
1. **Structure code for testability** — Put all pure logic (parsing, state machines, algorithms) in a library crate. Put hardware interaction in a binary crate that uses the library.
2. **Use `defmt` + `probe-rs` for real hardware** — `defmt` is a deferred formatting framework that is faster than semihosting and produces smaller binaries.
3. **Run host tests in CI, QEMU tests for integration** — Host tests catch logic bugs quickly. QEMU tests verify startup, memory layout, and interrupt handling.

## Summary
- QEMU emulates Cortex-M boards for hardware-free testing.
- Semihosting (`hprintln!`) bridges firmware output to the host terminal.
- Separate pure logic from hardware code and test logic on the host with `cargo test`.
- Use `defmt` + `probe-rs` for fast debug output on real hardware.
- Gate semihosting behind debug flags to avoid production hangs.

## Code Examples

**Testable firmware structure — cfg_attr makes the crate no_std for the target but allows cargo test on the host for pure logic**

```rust
// Testable embedded code structure
// This file compiles as no_std for the target and as std for host tests
#![cfg_attr(not(test), no_std)]
#![cfg_attr(not(test), no_main)]

// Pure logic module — no hardware dependencies
pub mod sensor {
    pub struct SensorReading {
        pub temp: u16,
        pub humidity: u16,
    }

    pub fn convert_temperature(raw: u16) -> f32 {
        (raw as f32 / 256.0) - 40.0
    }

    pub fn parse_packet(data: &[u8]) -> Option<SensorReading> {
        if data.len() < 4 { return None; }
        Some(SensorReading {
            temp: u16::from_be_bytes([data[0], data[1]]),
            humidity: u16::from_be_bytes([data[2], data[3]]),
        })
    }
}

// Host-side tests — run with `cargo test`
#[cfg(test)]
mod tests {
    use super::sensor::*;

    #[test]
    fn temp_at_zero_celsius() {
        // 40.0 * 256 = 10240 raw counts represents 0 degrees C
        assert_eq!(convert_temperature(10240), 0.0);
    }

    #[test]
    fn valid_packet_parses() {
        let data = [0x28, 0x00, 0x50, 0x00];
        let reading = parse_packet(&data).unwrap();
        assert_eq!(reading.temp, 0x2800);
        assert_eq!(reading.humidity, 0x5000);
    }

    #[test]
    fn short_packet_rejected() {
        assert!(parse_packet(&[0x00]).is_none());
    }
}
```


## Resources

- [defmt - efficient embedded logging](https://defmt.ferrous-systems.com/) — Deferred formatting framework for fast, small-footprint debug output
- [probe-rs](https://probe.rs/) — Modern embedded debugging toolkit with RTT support

---

> 📘 *This lesson is part of the [Rust for Embedded Systems](https://stanza.dev/courses/rust-embedded) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*