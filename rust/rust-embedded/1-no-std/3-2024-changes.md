---
source_course: "rust-embedded"
source_lesson: "rust-embedded-edition-2024-changes"
---

# Rust Edition 2024: What Embedded Developers Must Know

## Introduction
Rust Edition 2024 (stabilized in Rust 1.85) introduced several breaking changes that hit embedded code especially hard. If you are starting a new embedded project or upgrading an existing one, you need to understand `unsafe extern`, `#[unsafe(no_mangle)]`, the `static mut` deprecation, and the new `unsafe_op_in_unsafe_fn` lint.

## Key Concepts
- **`unsafe extern` blocks**: All `extern` blocks must now be marked `unsafe extern` because declaring foreign functions is inherently unsafe.
- **`#[unsafe(no_mangle)]`**: The `no_mangle` attribute must be wrapped in `unsafe()` because it can cause ABI mismatches.
- **`unsafe_op_in_unsafe_fn`**: The body of an `unsafe fn` is no longer implicitly an unsafe context — the compiler warns on implicit unsafe operations, and you should add explicit `unsafe {}` blocks inside.
- **`static mut` deprecation**: Creating references to `static mut` is denied by default because it enables trivial undefined behavior through aliasing.

## Real World Context
Embedded Rust code is full of `extern "C"` blocks, `#[no_mangle]` exports, `unsafe fn` interrupt handlers, and `static mut` shared state. Edition 2024 forces you to be explicit about every unsafe operation — this catches real bugs in interrupt-driven firmware where data races are common.

## Deep Dive

### unsafe extern blocks

Before Edition 2024, you wrote:

```rust
// Edition 2021 — compiles but hides unsafety
extern "C" {
    fn hardware_init();
}
```

Since Edition 2024, the `extern` block itself must be marked `unsafe`:

```rust
// Edition 2024 — explicit about the unsafety
unsafe extern "C" {
    fn hardware_init();
}
```

This makes it clear that declaring a foreign function is an unsafe contract — the compiler cannot verify that `hardware_init` actually exists with that signature.

### #[unsafe(no_mangle)]

The `no_mangle` attribute is used extensively in embedded code for entry points and interrupt vectors:

```rust
// Edition 2024 — wrap in unsafe()
#[unsafe(no_mangle)]
pub unsafe extern "C" fn Reset() -> ! {
    unsafe { main() }
}
```

The old `#[no_mangle]` form is rejected in Edition 2024.

### unsafe_op_in_unsafe_fn

Previously, the body of an `unsafe fn` was implicitly unsafe. Now you must be explicit:

```rust
// Edition 2024 — body is NOT implicitly unsafe
unsafe fn read_register(addr: *const u32) -> u32 {
    // Must wrap unsafe operations explicitly
    unsafe { core::ptr::read_volatile(addr) }
}
```

This is a big improvement for embedded code because it forces you to mark exactly which operations are unsafe, making audits much easier.

### static mut deprecation

The classic embedded pattern of sharing data via `static mut` is now denied:

```rust
// DENIED in Edition 2024 — creating references to static mut is UB-prone
static mut COUNTER: u32 = 0;

fn increment() {
    unsafe { COUNTER += 1; } // Error: creating a reference to static mut
}
```

Use atomics or a `Mutex<RefCell<>>` pattern instead:

```rust
use core::sync::atomic::{AtomicU32, Ordering};

// Safe alternative — no references to mutable statics
static COUNTER: AtomicU32 = AtomicU32::new(0);

fn increment() {
    COUNTER.fetch_add(1, Ordering::Relaxed);
}
```

For complex types that cannot be atomic, use `cortex_m::interrupt::Mutex<RefCell<T>>`.

## Common Pitfalls
1. **Migrating old code with `static mut` shared between ISR and main** — You cannot simply add `unsafe {}` wrappers; you must restructure to use atomics or critical-section-protected Mutex.
2. **Forgetting `unsafe {}` inside `unsafe fn`** — The compiler now emits warnings for every raw pointer dereference, volatile access, or FFI call inside an unsafe function body.
3. **Using `#[no_mangle]` from outdated tutorials** — Most embedded Rust tutorials were written for Edition 2021. Always translate `#[no_mangle]` to `#[unsafe(no_mangle)]`.

## Best Practices
1. **Set `edition = "2024"` in new projects** — Do not start new projects on Edition 2021. The Edition 2024 rules catch real bugs.
2. **Replace all `static mut` with atomics** — For simple counters and flags, `AtomicU32` / `AtomicBool` are zero-cost and safe. For complex types, use `Mutex<RefCell<Option<T>>>`.
3. **Audit unsafe blocks granularly** — Now that `unsafe fn` bodies require explicit blocks, take the opportunity to document the safety invariant for each `unsafe {}` block with a `// SAFETY:` comment.

## Summary
- `extern "C" { ... }` must become `unsafe extern "C" { ... }`.
- `#[no_mangle]` must become `#[unsafe(no_mangle)]`.
- `unsafe fn` bodies now warn on implicit unsafe operations — use explicit `unsafe {}` blocks.
- `static mut` references are denied — use atomics or `Mutex<RefCell<T>>`.
- These changes catch real bugs in interrupt-driven embedded code.

## Code Examples

**Edition 2024 embedded skeleton — shows unsafe extern, #[unsafe(no_mangle)], explicit unsafe blocks in unsafe fn, and AtomicU32 replacing static mut**

```rust
#![no_std]
#![no_main]

use core::panic::PanicInfo;
use core::sync::atomic::{AtomicU32, Ordering};

// Edition 2024: static mut replaced with atomic
static TICK_COUNT: AtomicU32 = AtomicU32::new(0);

// Edition 2024: #[unsafe(no_mangle)] and unsafe extern
#[unsafe(no_mangle)]
pub unsafe extern "C" fn SysTick() {
    TICK_COUNT.fetch_add(1, Ordering::Relaxed);
}

// Edition 2024: unsafe fn body requires explicit unsafe blocks
unsafe fn read_device_id() -> u32 {
    let id_reg: *const u32 = 0xE004_2000 as *const u32;
    // SAFETY: This address is the device ID register on STM32F4
    unsafe { core::ptr::read_volatile(id_reg) }
}

#[panic_handler]
fn panic(_info: &PanicInfo) -> ! {
    loop {}
}

#[unsafe(no_mangle)]
pub unsafe extern "C" fn _start() -> ! {
    let device_id = unsafe { read_device_id() };
    loop {
        let ticks = TICK_COUNT.load(Ordering::Relaxed);
        cortex_m::asm::nop();
    }
}
```


## Resources

- [Rust Edition 2024 Guide](https://doc.rust-lang.org/edition-guide/rust-2024/) — Official guide to all changes in the 2024 edition

---

> 📘 *This lesson is part of the [Rust for Embedded Systems](https://stanza.dev/courses/rust-embedded) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*