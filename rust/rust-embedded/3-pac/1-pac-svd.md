---
source_course: "rust-embedded"
source_lesson: "rust-embedded-pac-svd"
---

# Peripheral Access Crates: Typed Register Access

## Introduction
Microcontrollers expose their peripherals through memory-mapped registers — fixed addresses where reading and writing bits configures hardware. Peripheral Access Crates (PACs) are auto-generated Rust wrappers around these registers, giving you type-safe access instead of raw pointer arithmetic.

## Key Concepts
- **SVD (System View Description)**: An XML file from the chip vendor describing every peripheral register, its fields, bit positions, and access permissions.
- **svd2rust**: A tool that converts SVD files into Rust code with typed register access.
- **PAC API**: Three main operations — `read()`, `write()`, and `modify()` — each with type-safe field accessors.

## Real World Context
Without a PAC, configuring a UART means writing magic numbers to memory addresses — a single wrong bit crashes the system with no compiler help. With a PAC, the compiler rejects invalid field values and documents every register field as a method name.

## Deep Dive

### What SVD Files Contain

An SVD file describes the full register map of a microcontroller. It includes peripheral names, base addresses, register offsets, field bit ranges, and enumerated values. ARM CMSIS provides SVD files for most Cortex-M chips.

### Generating a PAC

The `svd2rust` tool converts an SVD file into a Rust crate:

```bash
svd2rust -i STM32F411.svd
```

This produces a full Rust crate with typed register access. Most developers use pre-generated PAC crates from crates.io (e.g., `stm32f4`, `nrf52840-pac`) rather than running svd2rust themselves.

### PAC API Patterns

The generated code provides three core operations:

```rust
use stm32f4::stm32f411;

let dp = stm32f411::Peripherals::take().unwrap();

// Read: returns a reader with field accessors
let is_high = dp.GPIOA.idr.read().idr5().bit();

// Write: replaces the entire register value
dp.GPIOA.odr.write(|w| w.odr5().set_bit());

// Modify: read-modify-write, change specific fields
dp.GPIOA.moder.modify(|_, w| {
    w.moder5().output()
     .moder6().input()
});
```

The `read()` method returns a reader struct with accessor methods for each field. The `write()` closure receives a writer that resets the register to its default value first. The `modify()` closure receives both a reader (current value) and a writer, allowing you to change specific fields without touching others.

### Type Safety

The PAC only allows valid values for each field:

```rust
dp.GPIOA.moder.modify(|_, w| {
    w.moder5().input()     // Binary 00 — valid
     .moder5().output()    // Binary 01 — valid
     .moder5().alternate() // Binary 10 — valid
     .moder5().analog()    // Binary 11 — valid
    // .moder5().bits(5)   // Would not compile — only 2-bit values allowed
});
```

This eliminates a huge class of bugs where a wrong bit pattern silently misconfigures hardware.

## Common Pitfalls
1. **Forgetting to enable peripheral clocks** — Most peripherals on ARM chips are clock-gated. You must enable the clock in the RCC peripheral before accessing any registers. Without it, reads return zero and writes are silently dropped.
2. **Using `write()` when you mean `modify()`** — `write()` resets all fields to their default value before applying your changes. If you only want to change one field, use `modify()` to preserve the others.
3. **Assuming SVD files are perfect** — Vendor SVD files often have errors (missing fields, wrong bit widths). Community-patched versions like `stm32-rs` fix known issues.

## Best Practices
1. **Use community-maintained PACs** — Crates like `stm32f4` (from `stm32-rs`) apply patches to fix SVD errors. Always prefer these over raw svd2rust output.
2. **Prefer HAL crates over raw PAC** — Use the PAC only when the HAL does not expose the functionality you need. HAL crates handle clock setup, pin configuration, and error handling.
3. **Read the reference manual alongside PAC docs** — The PAC method names match register field names from the reference manual. Use both together.

## Summary
- PACs are auto-generated from SVD files using svd2rust.
- Three core operations: `read()`, `write()`, and `modify()`.
- Type safety prevents invalid register field values at compile time.
- Enable peripheral clocks before accessing registers.
- Use community-maintained PAC crates from `stm32-rs` and similar projects.

## Code Examples

**PAC-level UART configuration showing RCC clock enable, baud rate register setup, and control register modification**

```rust
// Low-level UART setup using PAC register access
use stm32f4::stm32f411;

fn setup_uart(dp: &stm32f411::Peripherals) {
    // Step 1: Enable USART2 clock in the RCC peripheral
    dp.RCC.apb1enr.modify(|_, w| w.usart2en().enabled());

    // Step 2: Configure baud rate (assuming 84 MHz clock)
    // Baud = fck / (16 * USARTDIV)
    // For 115200: USARTDIV = 84_000_000 / (16 * 115200) = 45.57
    dp.USART2.brr.write(|w| unsafe {
        w.div_mantissa().bits(45)
         .div_fraction().bits(9)  // 0.5625 * 16 ~ 9
    });

    // Step 3: Enable transmitter, receiver, and the USART itself
    dp.USART2.cr1.modify(|_, w| {
        w.te().enabled()   // Transmitter enable
         .re().enabled()   // Receiver enable
         .ue().enabled()   // USART enable
    });
}

fn uart_write_byte(dp: &stm32f411::Peripherals, byte: u8) {
    // Wait until the transmit data register is empty
    while dp.USART2.sr.read().txe().bit_is_clear() {}
    // Write the byte
    dp.USART2.dr.write(|w| unsafe { w.dr().bits(byte as u16) });
}
```


## Resources

- [svd2rust documentation](https://docs.rs/svd2rust/) — Tool for generating Rust register access from SVD files
- [stm32-rs project](https://github.com/stm32-rs/stm32-rs) — Community-maintained, patched SVD files and PACs for STM32

---

> 📘 *This lesson is part of the [Rust for Embedded Systems](https://stanza.dev/courses/rust-embedded) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*