---
source_course: "rust-embedded"
source_lesson: "rust-embedded-no-std-basics"
---

# #![no_std]: Rust Without the Standard Library

## Introduction
The Rust standard library (`std`) assumes the presence of an operating system — it provides threads, file I/O, networking, and heap allocation. When you target bare-metal microcontrollers, there is no OS. The `#![no_std]` attribute tells the compiler to link only against `core`, the dependency-free foundation of Rust.

## Key Concepts
- **`#![no_std]`**: A crate-level attribute that opts out of the standard library, linking only `core`.
- **`#![no_main]`**: Tells the compiler not to expect a standard `main` entry point, since bare-metal targets use a custom entry.
- **`#[panic_handler]`**: A required function that defines what happens when `panic!` is called — there is no default on bare metal.
- **`core` crate**: The OS-independent foundation providing primitives, `Option`, `Result`, iterators, and atomics.

## Real World Context
Every embedded Rust firmware — from a blinking LED on a $2 STM32 to a satellite flight controller — starts with `#![no_std]`. Understanding what you lose and what you keep is the first step to writing reliable firmware.

## Deep Dive

A minimal bare-metal binary requires three things: `no_std`, `no_main`, and a panic handler. Here is the skeleton every embedded project starts from:

```rust
#![no_std]
#![no_main]

use core::panic::PanicInfo;

#[panic_handler]
fn panic(_info: &PanicInfo) -> ! {
    loop {}
}

#[unsafe(no_mangle)]
pub unsafe extern "C" fn _start() -> ! {
    loop {}
}
```

The `#[unsafe(no_mangle)]` attribute (required since Rust Edition 2024) prevents the compiler from mangling the symbol name so the linker can find `_start`. The `unsafe extern "C"` block uses the C calling convention.

Here is what you lose when you drop `std`, and what `core` still provides:

| std Feature | Status |
|-------------|--------|
| Collections (Vec, HashMap) | Lost (but see `alloc`) |
| I/O (println!, File) | Lost |
| Threads | Lost |
| Networking | Lost |
| Time (SystemTime) | Lost |

| core Feature | Available |
|--------------|----------|
| Primitives (i32, bool) | Yes |
| Option, Result | Yes |
| Iterators | Yes |
| Slices, arrays | Yes |
| Basic traits (Copy, Clone, Debug) | Yes |
| Atomics | Yes |
| core::mem, core::ptr | Yes |

In practice, most projects use the `cortex-m-rt` crate to handle the entry point boilerplate:

```rust
#![no_std]
#![no_main]

use cortex_m_rt::entry;
use panic_halt as _;

#[entry]
fn main() -> ! {
    // Your embedded code here
    loop {}
}
```

The `#[entry]` macro from `cortex-m-rt` sets up the vector table and calls your function after hardware initialization. The `panic_halt` crate provides a panic handler that simply halts the processor.

## Common Pitfalls
1. **Forgetting the panic handler** — Without `#[panic_handler]`, the linker will emit a cryptic error about a missing `rust_begin_unwind` symbol. Always include a panic handler crate like `panic-halt` or define your own.
2. **Using `#[no_mangle]` instead of `#[unsafe(no_mangle)]`** — Since Rust 1.85 (Edition 2024), the old `#[no_mangle]` form is rejected. Always use `#[unsafe(no_mangle)]`.
3. **Expecting `println!` to work** — `println!` requires `std`. Use `defmt`, `rtt-target`, or semihosting for debug output.

## Best Practices
1. **Use a panic handler crate** — `panic-halt`, `panic-rtt-target`, or `panic-probe` are battle-tested choices. Write a custom handler only if you need special behavior (e.g., blinking an error LED).
2. **Use `cortex-m-rt` for entry points** — Avoid writing raw `_start` symbols. The `#[entry]` macro handles vector table setup, memory initialization, and static variable placement.
3. **Separate pure logic from hardware** — Keep functions that do not touch hardware in a library crate that can be tested on the host with `cargo test`.

## Summary
- `#![no_std]` drops the standard library; you link only `core`.
- A bare-metal binary needs `#![no_main]` and a `#[panic_handler]`.
- `core` still gives you `Option`, `Result`, iterators, slices, and atomics.
- Use `cortex-m-rt` and a panic handler crate to avoid boilerplate.
- Since Edition 2024, use `#[unsafe(no_mangle)]` and `unsafe extern "C"`.

## Code Examples

**A minimal LED blinker using cortex-m-rt — the #[entry] macro handles vector table setup so you can focus on application logic**

```rust
#![no_std]
#![no_main]

use cortex_m_rt::entry;
use panic_halt as _;  // Panic handler that halts the processor

#[entry]
fn main() -> ! {
    // Get peripherals from the PAC
    let peripherals = stm32f4::Peripherals::take().unwrap();
    let gpioa = &peripherals.GPIOA;

    // Configure PA5 as push-pull output (LED on many STM32 boards)
    gpioa.moder.modify(|_, w| w.moder5().output());

    loop {
        // Toggle the LED by flipping the output data bit
        gpioa.odr.modify(|r, w| w.odr5().bit(!r.odr5().bit()));

        // Simple busy-wait delay
        for _ in 0..100_000 {
            cortex_m::asm::nop();
        }
    }
}
```


## Resources

- [The Embedded Rust Book](https://docs.rust-embedded.org/book/) — Official embedded Rust guide covering no_std, tooling, and hardware interaction
- [core crate documentation](https://doc.rust-lang.org/core/) — API reference for everything available without std

---

> 📘 *This lesson is part of the [Rust for Embedded Systems](https://stanza.dev/courses/rust-embedded) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*