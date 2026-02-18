---
source_course: "rust-embedded"
source_lesson: "rust-embedded-hal-implementations"
---

# Platform HAL Crates

## Common HAL Crates

| Platform | HAL Crate |
|----------|----------|
| STM32F4 | `stm32f4xx-hal` |
| STM32F1 | `stm32f1xx-hal` |
| nRF52 | `nrf52840-hal` |
| RP2040 | `rp2040-hal` |
| ESP32 | `esp-hal` |
| AVR | `avr-hal` |

## Layers of Abstraction

### 1. PAC (Peripheral Access Crate)

Raw register access, auto-generated from SVD:

```rust
// Direct register manipulation
peripherals.GPIOA.odr.modify(|_, w| w.odr5().set_bit());
```

### 2. HAL (Hardware Abstraction Layer)

Safe, ergonomic APIs:

```rust
use stm32f4xx_hal::{pac, prelude::*, gpio::*};

let dp = pac::Peripherals::take().unwrap();
let gpioa = dp.GPIOA.split();

// Type-safe pin configuration
let mut led = gpioa.pa5.into_push_pull_output();
led.set_high();
```

### 3. BSP (Board Support Package)

Board-specific convenience:

```rust
use stm32f4_discovery::Board;

let board = Board::new();
board.leds.ld3.on();
board.button.is_pressed();
```

## Choosing Pins at Compile Time

```rust
use stm32f4xx_hal::gpio::{PA5, Output, PushPull};

struct MyDevice {
    led: PA5<Output<PushPull>>,  // Type encodes pin configuration!
}

// Can't accidentally use wrong pin or mode
```

## Code Examples

**Complete HAL example**

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
    
    // Configure clocks
    let rcc = dp.RCC.constrain();
    let clocks = rcc.cfgr.sysclk(84.MHz()).freeze();
    
    // Configure GPIO
    let gpioa = dp.GPIOA.split();
    let mut led = gpioa.pa5.into_push_pull_output();
    
    // Configure timer
    let mut timer = Timer::syst(cp.SYST, &clocks).counter_hz();
    timer.start(1.Hz()).unwrap();
    
    loop {
        led.toggle();
        nb::block!(timer.wait()).unwrap();
    }
}
```


---

> 📘 *This lesson is part of the [Rust for Embedded Systems](https://stanza.dev/courses/rust-embedded) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*