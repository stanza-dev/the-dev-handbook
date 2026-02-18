---
source_course: "rust-embedded"
source_lesson: "rust-embedded-volatile"
---

# Memory-Mapped I/O

Hardware registers are memory addresses with side effects.

## The Problem

```rust
// This might be optimized away!
let status = unsafe { *(0x4002_0000 as *const u32) };
let status = unsafe { *(0x4002_0000 as *const u32) };
// Compiler: "Same address, same value - keep only one read"

// But hardware might change between reads!
```

## Volatile: Don't Optimize Away

```rust
use core::ptr::{read_volatile, write_volatile};

unsafe {
    // Every read actually reads
    let status = read_volatile(0x4002_0000 as *const u32);
    let status = read_volatile(0x4002_0000 as *const u32);
    
    // Every write actually writes
    write_volatile(0x4002_0004 as *mut u32, 0xFF);
}
```

## Why PACs Handle This

```rust
// PAC-generated code uses volatile internally
dp.GPIOA.idr.read();  // Uses read_volatile
dp.GPIOA.odr.write(|w| w.bits(0xFF));  // Uses write_volatile
```

## volatile_register Crate

```rust
use volatile_register::{RO, RW, WO};

#[repr(C)]
struct UartRegisters {
    dr: RW<u32>,     // Data register (read/write)
    sr: RO<u32>,     // Status register (read-only)
    cr: RW<u32>,     // Control register
    brr: WO<u32>,    // Baud rate (write-only)
}

let uart = unsafe { &*(0x4000_0000 as *const UartRegisters) };

uart.dr.write(0x55);        // OK
let status = uart.sr.read(); // OK
// uart.sr.write(0);         // Compile error! Read-only
// let _ = uart.brr.read();  // Compile error! Write-only
```

## Code Examples

**Manual volatile register access**

```rust
use core::ptr::{read_volatile, write_volatile};

// Manual register access
const GPIO_BASE: usize = 0x4002_0000;

#[repr(C)]
struct GpioRegs {
    moder: u32,   // Mode register
    otyper: u32,  // Output type
    ospeedr: u32, // Speed
    pupdr: u32,   // Pull-up/down
    idr: u32,     // Input data
    odr: u32,     // Output data
    bsrr: u32,    // Bit set/reset
}

fn gpio() -> &'static mut GpioRegs {
    unsafe { &mut *(GPIO_BASE as *mut GpioRegs) }
}

fn set_pin(pin: u8) {
    let regs = gpio();
    unsafe {
        // BSRR: write 1 to set, write to upper half to reset
        write_volatile(&mut regs.bsrr, 1 << pin);
    }
}

fn clear_pin(pin: u8) {
    let regs = gpio();
    unsafe {
        write_volatile(&mut regs.bsrr, 1 << (pin + 16));
    }
}

fn read_pin(pin: u8) -> bool {
    let regs = gpio();
    unsafe {
        (read_volatile(&regs.idr) >> pin) & 1 != 0
    }
}
```


---

> 📘 *This lesson is part of the [Rust for Embedded Systems](https://stanza.dev/courses/rust-embedded) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*