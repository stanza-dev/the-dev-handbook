---
source_course: "rust-embedded"
source_lesson: "rust-embedded-hal-implementations"
---

# Platform HAL Crates: PAC, HAL, and BSP

## Introduction
The `embedded-hal` traits are just interfaces — they need concrete implementations for each microcontroller family. These implementations come in three layers: PAC (raw registers), HAL (safe Rust API), and BSP (board-specific convenience). Understanding these layers helps you choose the right abstraction level for your project.

## Key Concepts
- **PAC (Peripheral Access Crate)**: Auto-generated from SVD files, provides typed register access with zero overhead.
- **HAL (Hardware Abstraction Layer)**: Safe, ergonomic API built on top of the PAC that implements `embedded-hal` traits.
- **BSP (Board Support Package)**: Board-specific crate mapping peripherals to physical pin names (e.g., `board.led`, `board.button`).

## Real World Context
When you buy an STM32F4 Discovery board and want to blink its LED, you have three choices: write raw register values through the PAC (tedious, error-prone), use the HAL for a safe API (recommended), or use the BSP for maximum convenience (if one exists for your exact board).

## Deep Dive

### Common HAL Crates

| Platform | HAL Crate |
|----------|----------|
| STM32F4 | `stm32f4xx-hal` |
| STM32F1 | `stm32f1xx-hal` |
| nRF52 | `nrf52840-hal` |
| RP2040 | `rp2040-hal` |
| ESP32 | `esp-hal` |
| AVR | `avr-hal` |

### Layer 1: PAC — Raw Register Access

PACs are auto-generated from SVD (System View Description) XML files. They give you typed access to every register and bit field:

```rust
// Direct register manipulation — functional but verbose
peripherals.GPIOA.moder.modify(|_, w| w.moder5().output());
peripherals.GPIOA.odr.modify(|_, w| w.odr5().set_bit());
```

This is low-level but type-safe: you cannot write an invalid value to a field.

### Layer 2: HAL — Safe, Ergonomic API

HAL crates wrap the PAC in safe abstractions and implement `embedded-hal` traits:

```rust
use stm32f4xx_hal::{pac, prelude::*, gpio::*};

let dp = pac::Peripherals::take().unwrap();
let gpioa = dp.GPIOA.split();

// Type-safe pin configuration
let mut led = gpioa.pa5.into_push_pull_output();
led.set_high();  // Uses OutputPin trait
```

Notice how the pin type `PA5<Output<PushPull>>` encodes the configuration at the type level. You cannot read from an output pin or write to an input pin — the compiler prevents it.

### Layer 3: BSP — Board Convenience

BSPs map HAL types to physical board features:

```rust
use stm32f4_discovery::Board;

let board = Board::new();
board.leds.ld3.on();       // Green LED by name
board.button.is_pressed();  // User button
```

BSPs are the easiest to use but the least portable — they are specific to one board.

### Type-State Pin Configuration

Rust's type system encodes pin state at compile time:

```rust
use stm32f4xx_hal::gpio::{PA5, Output, PushPull, Input, PullUp};

struct LedController {
    led: PA5<Output<PushPull>>,  // Type proves this is an output
}

// Attempting to pass an input pin here would be a compile error
```

This eliminates an entire class of runtime bugs where the wrong pin mode is used.

## Common Pitfalls
1. **Using the PAC directly when a HAL exists** — PAC code is verbose and error-prone. Always check if a HAL crate exists for your chip before writing raw register code.
2. **Forgetting to enable peripheral clocks** — Most peripherals are clock-gated by default. The HAL usually handles this in `split()` or `constrain()`, but at the PAC level you must enable clocks manually.
3. **Mixing pin types** — Once a pin is configured as output, you cannot use it as input without reconfiguring. The type system enforces this, so read the compiler errors carefully.

## Best Practices
1. **Start with the HAL, drop to PAC only when needed** — The HAL covers 90% of use cases. Only use the PAC for unsupported peripherals or performance-critical register access.
2. **Use `split()` to consume peripheral singletons** — This pattern (e.g., `dp.GPIOA.split()`) ensures each peripheral can only be configured once, preventing conflicting setups.
3. **Check for BSP crates for your board** — A BSP saves time during prototyping. Switch to HAL-level code when you design a custom PCB.

## Summary
- PAC: Auto-generated raw register access — type-safe but verbose.
- HAL: Safe API implementing `embedded-hal` traits — recommended for most code.
- BSP: Board-specific convenience — fastest to start but least portable.
- Pin types encode configuration at compile time, preventing mode errors.
- Always use `split()` and `constrain()` to safely claim peripherals.

## Code Examples

**Complete HAL-level application showing clock setup, GPIO configuration with type-state pins, and timer-based delay**

```rust
#![no_std]
#![no_main]

use cortex_m_rt::entry;
use stm32f4xx_hal::{pac, prelude::*, timer::Timer};
use panic_halt as _;

#[entry]
fn main() -> ! {
    let dp = pac::Peripherals::take().unwrap();
    let cp = cortex_m::Peripherals::take().unwrap();

    // Configure clocks — the HAL handles RCC register setup
    let rcc = dp.RCC.constrain();
    let clocks = rcc.cfgr.sysclk(84.MHz()).freeze();

    // Configure GPIO — split() claims GPIOA and enables its clock
    let gpioa = dp.GPIOA.split();
    let mut led = gpioa.pa5.into_push_pull_output();

    // Configure timer — wraps SysTick in a countdown interface
    let mut timer = Timer::syst(cp.SYST, &clocks).counter_hz();
    timer.start(1.Hz()).unwrap();

    loop {
        led.toggle();  // OutputPin::toggle from embedded-hal
        nb::block!(timer.wait()).unwrap();
    }
}
```


## Resources

- [stm32f4xx-hal documentation](https://docs.rs/stm32f4xx-hal/) — HAL implementation for the STM32F4 family

---

> 📘 *This lesson is part of the [Rust for Embedded Systems](https://stanza.dev/courses/rust-embedded) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*