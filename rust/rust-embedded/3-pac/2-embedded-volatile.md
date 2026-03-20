---
source_course: "rust-embedded"
source_lesson: "rust-embedded-volatile"
---

# Memory-Mapped I/O and Volatile Access

## Introduction
Hardware registers are not ordinary memory — reading a status register may clear a flag, writing a control register triggers an action. The compiler does not know this and may optimize away "redundant" reads or reorder writes. Volatile operations tell the compiler: every access matters, do not optimize it away.

## Key Concepts
- **Memory-mapped I/O (MMIO)**: Hardware peripherals are accessed through fixed memory addresses where reads and writes have side effects.
- **Volatile**: A qualifier that prevents the compiler from optimizing away, reordering, or coalescing memory accesses.
- **`read_volatile` / `write_volatile`**: Core library functions for performing volatile memory access through raw pointers.

## Real World Context
When polling a UART status register for incoming data, you read the same address repeatedly. Without volatile, the compiler sees multiple reads of the same address and keeps only the first — your firmware hangs forever waiting for data that already arrived.

## Deep Dive

### The Optimization Problem

The compiler applies dead-store elimination and redundant-load removal:

```rust
// Without volatile, the compiler may optimize this to a single read
let status = unsafe { *(0x4002_0000 as *const u32) };
let status = unsafe { *(0x4002_0000 as *const u32) };
// Compiler thinks: same address, same value — keep only one read
// But hardware may have changed the value between reads!
```

This is dangerous because hardware registers change asynchronously — an interrupt flag might set between reads, but the compiler eliminated the second read.

### Volatile: Every Access Counts

The `core::ptr` module provides volatile counterparts:

```rust
use core::ptr::{read_volatile, write_volatile};

unsafe {
    // Every read actually reaches the hardware
    let status1 = read_volatile(0x4002_0000 as *const u32);
    let status2 = read_volatile(0x4002_0000 as *const u32);
    // Both reads are preserved — status2 may differ from status1

    // Every write actually reaches the hardware
    write_volatile(0x4002_0004 as *mut u32, 0xFF);
}
```

Volatile guarantees the compiler emits a load/store instruction for every call, in the exact order written.

### Why PACs Handle This Automatically

PAC-generated code uses volatile internally, so you do not need to think about it:

```rust
// PAC read/write use volatile under the hood
dp.GPIOA.idr.read();                    // Uses read_volatile
dp.GPIOA.odr.write(|w| w.bits(0xFF));   // Uses write_volatile
```

This is one of the key benefits of PACs over raw pointer access.

### The volatile-register Crate

For custom MMIO layouts without a PAC, the `volatile-register` crate provides typed wrappers:

```rust
use volatile_register::{RO, RW, WO};

#[repr(C)]
struct UartRegisters {
    dr: RW<u32>,     // Data register — read/write
    sr: RO<u32>,     // Status register — read-only
    cr: RW<u32>,     // Control register — read/write
    brr: WO<u32>,    // Baud rate — write-only
}

let uart = unsafe { &*(0x4000_0000 as *const UartRegisters) };

uart.dr.write(0x55);         // OK — RW type
let status = uart.sr.read(); // OK — RO type
// uart.sr.write(0);         // Compile error: RO has no write method
// let _ = uart.brr.read();  // Compile error: WO has no read method
```

This combines volatile semantics with compile-time access-permission checks.

## Common Pitfalls
1. **Forgetting volatile on status polling loops** — A `while register == 0 {}` loop without volatile may compile into an infinite loop because the compiler loads the value once and keeps checking that cached value.
2. **Using volatile for all memory** — Volatile is only needed for MMIO. Using it for regular variables prevents legitimate optimizations and slows down code.
3. **Assuming volatile prevents reordering across peripherals** — Volatile only prevents reordering of accesses to the same address. For cross-peripheral ordering, use memory barriers (`core::sync::atomic::fence`).

## Best Practices
1. **Use PACs instead of manual volatile** — PAC-generated code handles volatile correctly. Only use raw `read_volatile` / `write_volatile` for custom hardware or registers not in the SVD.
2. **Use `volatile-register` for custom peripheral structs** — When writing a driver for custom hardware, `volatile-register` gives you type safety and volatile semantics.
3. **Add memory barriers when ordering matters** — If you need a write to peripheral A to complete before reading peripheral B, use `core::sync::atomic::compiler_fence(Ordering::SeqCst)`.

## Summary
- Hardware registers need volatile access because reads/writes have side effects.
- `read_volatile` and `write_volatile` prevent the compiler from optimizing away or reordering accesses.
- PACs use volatile internally, so prefer them over raw pointers.
- `volatile-register` provides `RO`, `RW`, `WO` wrappers for custom MMIO structures.
- Volatile does not replace memory barriers for cross-peripheral ordering.

## Code Examples

**Custom UART driver using volatile-register crate — RO/RW/WO types enforce access permissions at compile time while guaranteeing volatile semantics**

```rust
use volatile_register::{RO, RW, WO};

/// Memory-mapped UART register block for custom hardware
#[repr(C)]
struct UartRegs {
    dr: RW<u32>,     // Offset 0x00: Data register
    sr: RO<u32>,     // Offset 0x04: Status register
    cr: RW<u32>,     // Offset 0x08: Control register
    brr: WO<u32>,    // Offset 0x0C: Baud rate register
}

const UART_BASE: usize = 0x4000_4400;

fn uart_regs() -> &'static UartRegs {
    // SAFETY: UART_BASE is the documented base address for USART2
    unsafe { &*(UART_BASE as *const UartRegs) }
}

fn send_byte(byte: u8) {
    let regs = uart_regs();
    // Volatile read: actually checks hardware each iteration
    while regs.sr.read() & (1 << 7) == 0 {
        // Wait for TXE (transmit empty) flag
    }
    // Volatile write: actually sends the byte to hardware
    unsafe { regs.dr.write(byte as u32); }
}
```


## Resources

- [volatile-register crate](https://docs.rs/volatile-register/) — Typed volatile register access with RO/RW/WO wrappers

---

> 📘 *This lesson is part of the [Rust for Embedded Systems](https://stanza.dev/courses/rust-embedded) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*