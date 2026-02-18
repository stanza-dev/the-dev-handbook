---
source_course: "rust-embedded"
source_lesson: "rust-embedded-interrupt-handlers"
---

# Interrupts in Embedded Rust

Interrupts are asynchronous hardware events that preempt normal code.

## Defining Handlers

```rust
use cortex_m_rt::interrupt;
use stm32f4::stm32f411::interrupt as Interrupt;

#[interrupt]
fn TIM2() {
    // Handle timer interrupt
    // Runs when TIM2 fires
}

#[interrupt]
fn USART2() {
    // Handle UART interrupt
}
```

## The Problem: Shared Data

```rust
static mut COUNTER: u32 = 0;  // Mutable static!

#[interrupt]
fn TIM2() {
    unsafe {
        COUNTER += 1;  // Data race possible!
    }
}

fn main() -> ! {
    loop {
        unsafe {
            let c = COUNTER;  // May be corrupted!
        }
    }
}
```

## Solutions

### 1. Atomics

```rust
use core::sync::atomic::{AtomicU32, Ordering};

static COUNTER: AtomicU32 = AtomicU32::new(0);

#[interrupt]
fn TIM2() {
    COUNTER.fetch_add(1, Ordering::Relaxed);
}
```

### 2. Critical Sections

```rust
use cortex_m::interrupt::free;
use core::cell::RefCell;

static SHARED: cortex_m::interrupt::Mutex<RefCell<Option<Timer>>> =
    cortex_m::interrupt::Mutex::new(RefCell::new(None));

fn main() -> ! {
    free(|cs| {
        SHARED.borrow(cs).replace(Some(timer));
    });
    loop {}
}
```

### 3. RTIC (Real-Time Interrupt-driven Concurrency)

```rust
#[rtic::app(device = stm32f4::stm32f411)]
mod app {
    #[shared]
    struct Shared {
        counter: u32,
    }
    
    #[task(binds = TIM2, shared = [counter])]
    fn timer(cx: timer::Context) {
        *cx.shared.counter += 1;
    }
}
```

## Code Examples

**Safe interrupt handling**

```rust
// Safe interrupt sharing with Mutex
use cortex_m::interrupt::{free, Mutex};
use core::cell::RefCell;
use stm32f4::stm32f411::{self, TIM2};

static TIMER: Mutex<RefCell<Option<TIM2>>> = Mutex::new(RefCell::new(None));
static COUNTER: Mutex<RefCell<u32>> = Mutex::new(RefCell::new(0));

#[interrupt]
fn TIM2() {
    free(|cs| {
        // Increment counter
        let mut counter = COUNTER.borrow(cs).borrow_mut();
        *counter += 1;
        
        // Clear interrupt flag
        if let Some(timer) = TIMER.borrow(cs).borrow().as_ref() {
            timer.sr.modify(|_, w| w.uif().clear_bit());
        }
    });
}

fn main() -> ! {
    let dp = stm32f411::Peripherals::take().unwrap();
    
    // Setup timer and store in global
    free(|cs| {
        TIMER.borrow(cs).replace(Some(dp.TIM2));
    });
    
    // Enable interrupt
    unsafe {
        cortex_m::peripheral::NVIC::unmask(stm32f411::Interrupt::TIM2);
    }
    
    loop {
        free(|cs| {
            let count = *COUNTER.borrow(cs).borrow();
            // Use count
        });
    }
}
```


---

> 📘 *This lesson is part of the [Rust for Embedded Systems](https://stanza.dev/courses/rust-embedded) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*