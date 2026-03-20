---
source_course: "rust-embedded"
source_lesson: "rust-embedded-critical-sections"
---

# Critical Sections and the RTIC Framework

## Introduction
Critical sections (temporarily disabling interrupts) solve the data-sharing problem but scale poorly — in a complex system with many interrupt priorities, disabling all interrupts blocks time-critical handlers. RTIC (Real-Time Interrupt-driven Concurrency) is a framework that uses compile-time priority analysis to provide safe shared access with minimal latency.

## Key Concepts
- **Critical section**: A code region where interrupts are disabled, guaranteeing exclusive access to shared data.
- **`cortex_m::interrupt::free()`**: Disables all interrupts, runs a closure, then re-enables them.
- **RTIC**: A Rust framework that uses hardware interrupt priorities for safe, efficient resource sharing.
- **Priority-based locking**: RTIC temporarily raises the running priority to prevent preemption by tasks that share the same resource.

## Real World Context
A drone flight controller has tasks at different priorities: gyroscope sampling at 1 kHz (highest), PID loop at 500 Hz, telemetry at 10 Hz (lowest). Using `free()` for every shared access would block the gyroscope while telemetry runs — unacceptable for flight stability. RTIC ensures each task only blocks tasks of equal or lower priority.

## Deep Dive

### cortex_m::interrupt::free

The simplest critical section disables all maskable interrupts:

```rust
use cortex_m::interrupt::free;

let result = free(|cs| {
    // All interrupts disabled in this block
    // cs is a CriticalSection token proving this
    read_shared_data(cs)
});  // Interrupts automatically re-enabled
```

The `CriticalSection` token `cs` is zero-sized — it exists only to prove at the type level that interrupts are disabled. Types like `Mutex` require it to grant access.

### Keep Critical Sections Short

Disabling interrupts blocks all hardware events. Keep critical sections as short as possible:

```rust
// BAD: long critical section blocks all interrupts
free(|cs| {
    let data = SHARED.borrow(cs).borrow();
    expensive_computation(*data);  // Interrupts blocked during computation!
});

// GOOD: copy data out, process with interrupts enabled
let data = free(|cs| *SHARED.borrow(cs).borrow());
expensive_computation(data);  // Interrupts enabled!
```

The second pattern copies the value inside the critical section and processes it outside. This minimizes interrupt latency.

### RTIC v2: Priority-Based Concurrency

RTIC analyzes your task declarations at compile time and uses the NVIC priority mechanism for safe resource sharing:

```rust
#[rtic::app(device = stm32f4::stm32f411, dispatchers = [SPI1])]
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
    fn init(cx: init::Context) -> (Shared, Local) {
        let dp = cx.device;
        let rcc = dp.RCC.constrain();
        let clocks = rcc.cfgr.sysclk(84.MHz()).freeze();

        let gpioa = dp.GPIOA.split();
        let led = gpioa.pa5.into_push_pull_output();

        let mut timer = dp.TIM2.counter_hz(&clocks);
        timer.start(1.Hz()).unwrap();
        timer.listen(Event::Update);

        (Shared { counter: 0 }, Local { led, timer })
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
            cortex_m::asm::wfi();
        }
    }
}
```

Notice that `init` returns `(Shared, Local)` — RTIC v2 removed the `init::Monotonics` return value from v1. The `lock()` method on shared resources temporarily raises the running priority to prevent preemption, then restores it. This is faster than disabling all interrupts because higher-priority tasks can still run.

### RTIC vs Embassy

Both RTIC and Embassy provide safe concurrency but use different models:

| Feature | RTIC | Embassy |
|---------|------|---------|
| Scheduling | Priority-based preemption | Cooperative async |
| Shared data | `.lock()` with priority ceiling | Channels and signals |
| Task model | Hardware interrupts | Async futures |
| Timing | Monotonic clocks | embassy-time |

RTIC is better for hard real-time deadlines. Embassy is better for complex async workflows.

## Common Pitfalls
1. **Nesting `free()` calls** — Calling `free()` inside a `free()` block is redundant but harmless. However, storing the `CriticalSection` token and using it outside the closure is unsound.
2. **Confusing RTIC v1 and v2 syntax** — RTIC v2 changed the `init` return type (removed Monotonics), task attribute syntax, and resource access patterns. Use the v2 documentation.
3. **Deadlock with priority inversion** — If a low-priority task holds a lock that a high-priority task needs, the high-priority task is blocked. RTIC prevents this by design with priority ceiling protocol.

## Best Practices
1. **Use RTIC for multi-priority systems** — If you have more than two interrupt priorities, RTIC's compile-time analysis is safer and faster than manual critical sections.
2. **Use `free()` for simple systems** — If you have one or two interrupts sharing data with main, `free()` with `Mutex<RefCell<>>` is simpler than bringing in RTIC.
3. **Declare resources explicitly** — In RTIC, list every shared resource in the `#[shared]` struct. This gives the framework full visibility for priority ceiling computation.

## Summary
- `cortex_m::interrupt::free()` disables all interrupts for safe data access.
- Keep critical sections short — copy data out and process outside.
- RTIC v2 uses compile-time priority analysis for efficient, deadlock-free resource sharing.
- RTIC v2 `init` returns `(Shared, Local)` — no more `Monotonics` tuple.
- Choose RTIC for hard real-time, Embassy for async workflows.

## Code Examples

**Complete RTIC v2 application — init returns (Shared, Local), shared resources use lock() for priority-ceiling-based safe access**

```rust
// RTIC v2 application with shared and local resources
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
    fn init(cx: init::Context) -> (Shared, Local) {
        let dp = cx.device;
        let rcc = dp.RCC.constrain();
        let clocks = rcc.cfgr.sysclk(84.MHz()).freeze();

        let gpioa = dp.GPIOA.split();
        let led = gpioa.pa5.into_push_pull_output();

        let mut timer = dp.TIM2.counter_hz(&clocks);
        timer.start(1.Hz()).unwrap();
        timer.listen(Event::Update);

        (Shared { counter: 0 }, Local { led, timer })
    }

    // Hardware task: bound to TIM2 interrupt, shares counter with idle
    #[task(binds = TIM2, shared = [counter], local = [led, timer])]
    fn tick(mut cx: tick::Context) {
        cx.local.timer.clear_interrupt(Event::Update);
        cx.local.led.toggle();
        // lock() raises priority ceiling — safe, no interrupt disable
        cx.shared.counter.lock(|c| *c += 1);
    }

    #[idle(shared = [counter])]
    fn idle(mut cx: idle::Context) -> ! {
        loop {
            let count = cx.shared.counter.lock(|c| *c);
            cortex_m::asm::wfi(); // Sleep until next interrupt
        }
    }
}
```


## Resources

- [RTIC v2 Book](https://rtic.rs/2/book/en/) — RTIC v2 framework documentation and tutorials

---

> 📘 *This lesson is part of the [Rust for Embedded Systems](https://stanza.dev/courses/rust-embedded) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*