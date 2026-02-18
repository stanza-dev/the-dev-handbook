---
source_course: "rust-embedded"
source_lesson: "rust-embedded-no-std-basics"
---

# #![no_std]: Rust Without Standard Library

The std library requires an OS. For bare metal, use `no_std`.

```rust
#![no_std]
#![no_main]

use core::panic::PanicInfo;

#[panic_handler]
fn panic(_info: &PanicInfo) -> ! {
    loop {}
}

#[no_mangle]
pub extern "C" fn _start() -> ! {
    loop {}
}
```

## What You Lose

| std Feature | Status |
|-------------|--------|
| Collections (Vec, HashMap) | Lost (but see `alloc`) |
| I/O (println!, File) | Lost |
| Threads | Lost |
| Networking | Lost |
| Time (SystemTime) | Lost |

## What You Keep (core)

| core Feature | Available |
|--------------|----------|
| Primitives (i32, bool) | ✓ |
| Option, Result | ✓ |
| Iterators | ✓ |
| Slices, arrays | ✓ |
| Basic traits (Copy, Clone, Debug) | ✓ |
| Atomics | ✓ |
| core::mem, core::ptr | ✓ |

## Required Components

### Panic Handler

```rust
#[panic_handler]
fn panic(info: &PanicInfo) -> ! {
    // Can log, blink LED, etc.
    loop {} // Never return
}
```

### Entry Point

```rust
#[no_mangle]
pub extern "C" fn main() -> ! {
    // Your embedded code
    loop {}
}

// Or use cortex-m-rt
#[entry]
fn main() -> ! {
    loop {}
}
```

See [The Embedded Rust Book](https://docs.rust-embedded.org/book/).

## Code Examples

**Basic LED blinker**

```rust
#![no_std]
#![no_main]

use cortex_m_rt::entry;
use panic_halt as _;  // Panic handler that halts

#[entry]
fn main() -> ! {
    // Get peripherals
    let peripherals = stm32f4::Peripherals::take().unwrap();
    let gpioa = &peripherals.GPIOA;
    
    // Configure PA5 as output (LED on many boards)
    gpioa.moder.modify(|_, w| w.moder5().output());
    
    loop {
        // Toggle LED
        gpioa.odr.modify(|r, w| w.odr5().bit(!r.odr5().bit()));
        
        // Delay (busy wait)
        for _ in 0..100_000 {
            cortex_m::asm::nop();
        }
    }
}
```


## Resources

- [The Embedded Rust Book](https://docs.rust-embedded.org/book/) — Official embedded Rust guide

---

> 📘 *This lesson is part of the [Rust for Embedded Systems](https://stanza.dev/courses/rust-embedded) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*