---
source_course: "rust-embedded"
source_lesson: "rust-embedded-hal-traits"
---

# embedded-hal: Portable Drivers

The `embedded-hal` crate defines standard traits for hardware.

## Key Traits

### Digital I/O

```rust
use embedded_hal::digital::{OutputPin, InputPin, PinState};

// Any type implementing OutputPin
fn blink<P: OutputPin>(pin: &mut P) {
    pin.set_high().unwrap();
    delay();
    pin.set_low().unwrap();
}
```

### Delays

```rust
use embedded_hal::delay::DelayNs;

fn wait<D: DelayNs>(delay: &mut D) {
    delay.delay_ms(100);
}
```

### I2C

```rust
use embedded_hal::i2c::I2c;

fn read_sensor<I: I2c>(i2c: &mut I, addr: u8) -> Result<u16, I::Error> {
    let mut buf = [0u8; 2];
    i2c.write_read(addr, &[0x00], &mut buf)?;
    Ok(u16::from_be_bytes(buf))
}
```

### SPI

```rust
use embedded_hal::spi::SpiDevice;

fn transfer<S: SpiDevice>(spi: &mut S, data: &mut [u8]) -> Result<(), S::Error> {
    spi.transfer_in_place(data)?;
    Ok(())
}
```

## The Ecosystem

```
+------------------+
|   Your Code      |
+------------------+
         |
+------------------+
|  Device Drivers  |  (e.g., bme280, ssd1306)
+------------------+
         |
+------------------+
|  embedded-hal    |  (traits)
+------------------+
         |
+------------------+
|   HAL Crate      |  (e.g., stm32f4xx-hal)
+------------------+
         |
+------------------+
|      PAC         |  (e.g., stm32f4)
+------------------+
         |
+------------------+
|    Hardware      |
+------------------+
```

## Code Examples

**Portable I2C driver**

```rust
// Portable temperature sensor driver
use embedded_hal::i2c::I2c;

pub struct TemperatureSensor<I> {
    i2c: I,
    address: u8,
}

impl<I: I2c> TemperatureSensor<I> {
    pub fn new(i2c: I, address: u8) -> Self {
        TemperatureSensor { i2c, address }
    }
    
    pub fn read_temperature(&mut self) -> Result<f32, I::Error> {
        let mut buf = [0u8; 2];
        
        // Send read command
        self.i2c.write(self.address, &[0x00])?;
        
        // Read temperature
        self.i2c.read(self.address, &mut buf)?;
        
        // Convert to Celsius
        let raw = i16::from_be_bytes(buf);
        Ok(raw as f32 / 128.0)
    }
}

// Works on ANY platform with I2C!
// STM32:
let i2c = stm32f4xx_hal::i2c::I2c::new(...);
let mut sensor = TemperatureSensor::new(i2c, 0x48);

// nRF52:
let i2c = nrf52840_hal::twim::Twim::new(...);
let mut sensor = TemperatureSensor::new(i2c, 0x48);

// Same code!
```


## Resources

- [embedded-hal](https://docs.rs/embedded-hal/) — embedded-hal documentation

---

> 📘 *This lesson is part of the [Rust for Embedded Systems](https://stanza.dev/courses/rust-embedded) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*