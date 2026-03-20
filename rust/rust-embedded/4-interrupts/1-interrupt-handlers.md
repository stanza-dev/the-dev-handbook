---
source_course: "rust-embedded"
source_lesson: "rust-embedded-interrupt-handlers"
---

# Interrupts in Embedded Rust

## Introduction
Interrupts are asynchronous hardware events that preempt normal code execution — a timer expires, a byte arrives on UART, a button is pressed. Handling interrupts safely is the central challenge of embedded programming, and Rust's type system provides powerful tools to prevent the data races that plague C firmware.

## Key Concepts
- **Interrupt handler**: A function that runs when a specific hardware event occurs, preempting the currently executing code.
- **NVIC (Nested Vectored Interrupt Controller)**: The ARM hardware unit that manages interrupt priorities and enables/disables individual interrupt sources.
- **Data race**: When an interrupt handler and main code access the same data without synchronization, leading to corrupted values.
- **Atomics**: CPU-level operations that complete in a single instruction, immune to preemption.

## Real World Context
Almost every embedded system uses interrupts — a real-time clock ticking every millisecond, a sensor data-ready signal, a communication peripheral receiving bytes. Getting interrupt-shared data wrong causes intermittent bugs that are nearly impossible to reproduce.

## Deep Dive

### Defining Handlers

The `cortex-m-rt` crate provides the `#[interrupt]` attribute for declaring interrupt handlers:

```rust
use cortex_m_rt::interrupt;
use stm32f4::stm32f411::interrupt as Interrupt;

#[interrupt]
fn TIM2() {
    // Runs when TIM2 fires
    // Must be fast — blocks all lower-priority interrupts
}

#[interrupt]
fn USART2() {
    // Runs when USART2 has data or an event
}
```

The function name must exactly match the interrupt vector name from the PAC. The `#[interrupt]` macro sets up the vector table entry.

### The Shared Data Problem

The classic C approach — `static mut` — is now denied in Rust Edition 2024:

```rust
// DENIED in Edition 2024 — creating references to static mut is UB
static mut COUNTER: u32 = 0;

#[interrupt]
fn TIM2() {
    unsafe { COUNTER += 1; } // Error!
}
```

Even in older editions this was unsound: if main reads `COUNTER` while the interrupt handler is halfway through a 32-bit increment on an 8-bit MCU, the value is corrupted.

### Solution 1: Atomics

For simple values (counters, flags), use atomics:

```rust
use core::sync::atomic::{AtomicU32, Ordering};

static COUNTER: AtomicU32 = AtomicU32::new(0);

#[interrupt]
fn TIM2() {
    COUNTER.fetch_add(1, Ordering::Relaxed);
}

fn main_loop() {
    let count = COUNTER.load(Ordering::Relaxed);
    // Use count safely — atomic load is indivisible
}
```

Atomics are the simplest solution and have zero overhead on Cortex-M3+ (which has native atomic instructions). On Cortex-M0, atomics may disable interrupts briefly.

### Solution 2: Critical Sections with Mutex

For complex types that cannot be atomic (like peripheral handles), use `cortex_m::interrupt::Mutex` with `RefCell`:

```rust
use cortex_m::interrupt::{free, Mutex};
use core::cell::RefCell;

static SHARED_TIMER: Mutex<RefCell<Option<TIM2>>> =
    Mutex::new(RefCell::new(None));

#[interrupt]
fn TIM2() {
    free(|cs| {
        if let Some(timer) = SHARED_TIMER.borrow(cs).borrow().as_ref() {
            timer.sr.modify(|_, w| w.uif().clear_bit());
        }
    });
}
```

The `free()` function disables all interrupts for the duration of the closure, guaranteeing exclusive access. The `cs` (CriticalSection) token proves to the type system that interrupts are disabled.

## Common Pitfalls
1. **Long critical sections** — Disabling interrupts for too long causes missed events. Copy data out of the critical section and process it outside.
2. **Not clearing the interrupt flag** — Most peripherals keep firing the same interrupt until you clear the pending flag in the peripheral's status register. Failure to clear it causes an interrupt storm.
3. **Using `static mut` from old tutorials** — Edition 2024 denies `static mut` references. Migrate to atomics or `Mutex<RefCell<Option<T>>>`.

## Best Practices
1. **Use atomics for simple values** — Counters, flags, and small status values should always be `AtomicU32`, `AtomicBool`, etc.
2. **Keep interrupt handlers short** — Do the minimum work (read a value, set a flag, clear the interrupt), then process in the main loop.
3. **Use `Option<T>` for late-initialized peripherals** — Peripherals are created in `main()` but used in interrupt handlers. Wrap them in `Mutex<RefCell<Option<T>>>` and call `.replace(Some(peripheral))` during init.

## Summary
- Interrupt handlers are declared with `#[interrupt]` and must match PAC vector names.
- `static mut` is denied in Edition 2024 — use atomics or `Mutex<RefCell<>>`.
- Atomics are zero-overhead for simple types on Cortex-M3+.
- Critical sections (`free()`) disable interrupts for safe access to complex types.
- Always clear interrupt flags to prevent interrupt storms.

## Code Examples

**Safe interrupt handling combining atomics for simple values and Mutex<RefCell<Option<T>>> for peripheral access — no static mut needed**

```rust
use cortex_m::interrupt::{free, Mutex};
use core::cell::RefCell;
use core::sync::atomic::{AtomicU32, Ordering};
use stm32f4::stm32f411::{self, TIM2};

// Simple counter: use atomics (zero overhead)
static TICK_COUNT: AtomicU32 = AtomicU32::new(0);

// Complex peripheral: use Mutex<RefCell<Option<T>>>
static TIMER: Mutex<RefCell<Option<TIM2>>> =
    Mutex::new(RefCell::new(None));

#[interrupt]
fn TIM2() {
    // Atomic increment — safe, no critical section needed
    TICK_COUNT.fetch_add(1, Ordering::Relaxed);

    // Clear interrupt flag — requires critical section for peripheral access
    free(|cs| {
        if let Some(timer) = TIMER.borrow(cs).borrow().as_ref() {
            timer.sr.modify(|_, w| w.uif().clear_bit());
        }
    });
}

fn setup(dp: stm32f411::Peripherals) {
    // Store peripheral in global for ISR access
    free(|cs| {
        TIMER.borrow(cs).replace(Some(dp.TIM2));
    });

    // Enable interrupt in NVIC
    unsafe {
        cortex_m::peripheral::NVIC::unmask(stm32f411::Interrupt::TIM2);
    }
}
```


## Resources

- [The Embedded Rust Book - Concurrency](https://docs.rust-embedded.org/book/concurrency/) — Official guide to interrupt-safe concurrency in embedded Rust

---

> 📘 *This lesson is part of the [Rust for Embedded Systems](https://stanza.dev/courses/rust-embedded) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*