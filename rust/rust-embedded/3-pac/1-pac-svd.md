---
source_course: "rust-embedded"
source_lesson: "rust-embedded-pac-svd"
---

# Peripheral Access Crates

PACs are auto-generated from SVD (System View Description) files.

## What's an SVD File?

XML description of a microcontroller's:
- Peripheral registers
- Register fields
- Bit positions
- Access permissions (read/write/read-write)

## svd2rust: SVD to Rust

```bash
svd2rust -i STM32F411.svd
```

Generates type-safe register access:

```rust
// Instead of magic numbers:
unsafe { *(0x4002_0000 as *mut u32) = 1 << 5; }

// Type-safe API:
peripherals.GPIOA.moder.modify(|_, w| w.moder5().output());
```

## PAC API Pattern

```rust
use stm32f4::stm32f411;

let dp = stm32f411::Peripherals::take().unwrap();

// Read a register
let value = dp.GPIOA.idr.read().idr5().bit();

// Write entire register
dp.GPIOA.odr.write(|w| w.odr5().set_bit());

// Modify specific fields
dp.GPIOA.moder.modify(|_, w| {
    w.moder5().output()
     .moder6().input()
});

// Reset to default
dp.GPIOA.odr.reset();
```

## Type Safety

```rust
// Only valid values can be written
dp.GPIOA.moder.modify(|_, w| {
    w.moder5().input()          // 00
     .moder5().output()         // 01
     .moder5().alternate()      // 10
     .moder5().analog()         // 11
    // .moder5().invalid()      // Won't compile!
});
```

## Code Examples

**PAC-level UART**

```rust
// Low-level UART setup using PAC
use stm32f4::stm32f411;

fn setup_uart(dp: &stm32f411::Peripherals) {
    // Enable USART2 clock
    dp.RCC.apb1enr.modify(|_, w| w.usart2en().enabled());
    
    // Configure baud rate (assuming 84MHz clock)
    // Baud = fck / (16 * USARTDIV)
    // For 115200: USARTDIV = 84_000_000 / (16 * 115200) = 45.57
    dp.USART2.brr.write(|w| unsafe {
        w.div_mantissa().bits(45)
         .div_fraction().bits(9)  // 0.5625 * 16 ≈ 9
    });
    
    // Enable TX, RX, and USART
    dp.USART2.cr1.modify(|_, w| {
        w.te().enabled()   // Transmitter enable
         .re().enabled()   // Receiver enable
         .ue().enabled()   // USART enable
    });
}

fn uart_write_byte(dp: &stm32f411::Peripherals, byte: u8) {
    // Wait for TX empty
    while dp.USART2.sr.read().txe().bit_is_clear() {}
    
    // Write byte
    dp.USART2.dr.write(|w| unsafe { w.dr().bits(byte as u16) });
}

fn uart_read_byte(dp: &stm32f411::Peripherals) -> u8 {
    // Wait for RX not empty
    while dp.USART2.sr.read().rxne().bit_is_clear() {}
    
    // Read byte
    dp.USART2.dr.read().dr().bits() as u8
}
```


---

> 📘 *This lesson is part of the [Rust for Embedded Systems](https://stanza.dev/courses/rust-embedded) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*