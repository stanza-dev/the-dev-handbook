---
source_course: "rust-embedded"
source_lesson: "rust-embedded-critical-sections"
---

# Critical Sections

Disable interrupts temporarily to ensure atomicity.

## cortex_m::interrupt::free

```rust
use cortex_m::interrupt::free;

let result = free(|cs| {
    // Interrupts disabled in this block
    // cs is a CriticalSection token
    do_critical_work()
});  // Interrupts re-enabled
```

## The Mutex Pattern

```rust
use cortex_m::interrupt::Mutex;
use core::cell::RefCell;

// Global state protected by Mutex
static DATA: Mutex<RefCell<u32>> = Mutex::new(RefCell::new(0));

fn read_data() -> u32 {
    free(|cs| *DATA.borrow(cs).borrow())
}

fn write_data(value: u32) {
    free(|cs| *DATA.borrow(cs).borrow_mut() = value)
}
```

## Keep Critical Sections Short!

```rust
// Bad: long critical section
free(|cs| {
    let data = DATA.borrow(cs).borrow();
    expensive_computation(*data);  // Interrupts blocked!
});

// Good: copy and release
let data = free(|cs| *DATA.borrow(cs).borrow());
expensive_computation(data);  // Interrupts enabled
```

# RTIC: Real-Time Interrupt-driven Concurrency

Framework for safe, efficient embedded concurrency.

```rust
#[rtic::app(device = stm32f4::stm32f411, dispatchers = [SPI1])]
mod app {
    use super::*;
    
    #[shared]
    struct Shared {
        led: Led,
    }
    
    #[local]
    struct Local {
        button: Button,
    }
    
    #[init]
    fn init(cx: init::Context) -> (Shared, Local, init::Monotonics) {
        // Setup code
        (Shared { led }, Local { button }, init::Monotonics())
    }
    
    #[task(binds = EXTI0, shared = [led], local = [button])]
    fn button_press(mut cx: button_press::Context) {
        cx.shared.led.lock(|led| led.toggle());
    }
}
```

## Code Examples

**Complete RTIC application**

```rust
// RTIC example with shared resources
#[rtic::app(
    device = stm32f4::stm32f411,
    peripherals = true,
    dispatchers = [SPI1, SPI2]
)]
mod app {
    use stm32f4xx_hal::{prelude::*, gpio::*, timer::*};
    
    #[shared]
    struct Shared {
        counter: u32,
    }
    
    #[local]
    struct Local {
        led: PA5<Output<PushPull>>,
        timer: CounterHz<TIM2>,
    }
    
    #[init]
    fn init(cx: init::Context) -> (Shared, Local, init::Monotonics) {
        let dp = cx.device;
        let rcc = dp.RCC.constrain();
        let clocks = rcc.cfgr.sysclk(84.MHz()).freeze();
        
        let gpioa = dp.GPIOA.split();
        let led = gpioa.pa5.into_push_pull_output();
        
        let mut timer = dp.TIM2.counter_hz(&clocks);
        timer.start(1.Hz()).unwrap();
        timer.listen(Event::Update);
        
        (Shared { counter: 0 }, Local { led, timer }, init::Monotonics())
    }
    
    #[task(binds = TIM2, shared = [counter], local = [led, timer])]
    fn tick(mut cx: tick::Context) {
        cx.local.timer.clear_interrupt(Event::Update);
        cx.local.led.toggle();
        
        cx.shared.counter.lock(|c| {
            *c += 1;
        });
    }
    
    #[idle(shared = [counter])]
    fn idle(mut cx: idle::Context) -> ! {
        loop {
            let count = cx.shared.counter.lock(|c| *c);
            // Do something with count
            cortex_m::asm::wfi();  // Wait for interrupt
        }
    }
}
```


## Resources

- [RTIC Book](https://rtic.rs/) — RTIC framework documentation

---

> 📘 *This lesson is part of the [Rust for Embedded Systems](https://stanza.dev/courses/rust-embedded) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*