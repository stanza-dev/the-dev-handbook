---
source_course: "rust-embedded"
source_lesson: "rust-embedded-testing-qemu"
---

# Testing Embedded Code

## QEMU for Hardware Emulation

```bash
# Install QEMU for ARM
brew install qemu  # macOS
apt install qemu-system-arm  # Ubuntu

# Run firmware in emulator
qemu-system-arm \
    -cpu cortex-m4 \
    -machine lm3s6965evb \
    -nographic \
    -semihosting-config enable=on,target=native \
    -kernel target/thumbv7m-none-eabi/release/my-app
```

## Semihosting: Print from Embedded

```rust
use cortex_m_semihosting::hprintln;

#[entry]
fn main() -> ! {
    hprintln!("Hello from embedded!").unwrap();
    loop {}
}
```

## cargo-embed: Simplified Workflow

```bash
cargo install cargo-embed

# Flash and run
cargo embed --release
```

```toml
# Embed.toml
[default.general]
chip = "STM32F411RETx"

[default.rtt]
enabled = true

[default.gdb]
enabled = false
```

## Unit Tests on Host

```rust
// In lib.rs - test logic on host
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_parser() {
        let result = parse_sensor_data(&[0x12, 0x34]);
        assert_eq!(result, 0x1234);
    }
}
```

```bash
# Run tests on host machine
cargo test
```

## Integration Tests with probe-rs

```bash
cargo install probe-rs

# Flash and run with RTT output
cargo run --release
```

## Code Examples

**Testable code structure**

```rust
// Testable embedded code structure
#![cfg_attr(not(test), no_std)]
#![cfg_attr(not(test), no_main)]

// Pure logic - testable on host
mod sensor {
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
}

#[cfg(test)]
mod tests {
    use super::sensor::*;
    
    #[test]
    fn test_temperature_conversion() {
        assert_eq!(convert_temperature(0x2800), 0.0);  // 40°C raw = 0°C
        assert_eq!(convert_temperature(0x4000), 24.0); // ~24°C
    }
    
    #[test]
    fn test_packet_parsing() {
        let data = [0x28, 0x00, 0x50, 0x00];
        let reading = parse_packet(&data).unwrap();
        assert_eq!(reading.temp, 0x2800);
    }
    
    #[test]
    fn test_short_packet() {
        assert!(parse_packet(&[0x00, 0x01]).is_none());
    }
}

// Hardware-specific code - only on target
#[cfg(not(test))]
mod main {
    use cortex_m_rt::entry;
    
    #[entry]
    fn main() -> ! {
        // Use sensor:: functions here
        loop {}
    }
}
```


## Resources

- [probe-rs](https://probe.rs/) — Modern embedded debugging toolkit

---

> 📘 *This lesson is part of the [Rust for Embedded Systems](https://stanza.dev/courses/rust-embedded) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*