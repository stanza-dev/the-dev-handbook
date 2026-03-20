---
source_course: "rust-embedded"
source_lesson: "rust-embedded-target-triples"
---

# Cross-Compilation for Embedded Targets

## Introduction
When you run `cargo build` normally, Rust compiles for your host machine. Embedded firmware must be compiled for a completely different architecture — ARM Cortex-M, RISC-V, or even AVR. Cross-compilation in Rust is straightforward: install the target, configure Cargo, and build.

## Key Concepts
- **Target triple**: A string like `thumbv7em-none-eabihf` that identifies the architecture, vendor, OS, and ABI of the compilation target.
- **Cross-compilation**: Compiling code on one platform (e.g., x86_64 Linux) to run on another (e.g., ARM Cortex-M4).
- **Linker script (memory.x)**: A file that tells the linker where to place code and data in the microcontroller's memory map.

## Real World Context
Every embedded Rust project involves cross-compilation. Your development machine is x86_64 or aarch64, but the firmware runs on a Cortex-M0 or RISC-V. Understanding target triples and the build configuration is essential for getting anything to compile.

## Deep Dive

### Target Triple Format

A target triple follows the pattern:

```
<arch>-<vendor>-<os>-<abi>
```

For embedded targets, vendor is often `unknown` or omitted, and OS is `none` (bare metal).

### Common Embedded Targets

| Target | Description |
|--------|-------------|
| `thumbv7em-none-eabihf` | Cortex-M4F/M7F with hardware FPU |
| `thumbv7em-none-eabi` | Cortex-M4/M7 without FPU |
| `thumbv7m-none-eabi` | Cortex-M3 |
| `thumbv6m-none-eabi` | Cortex-M0/M0+ |
| `riscv32imac-unknown-none-elf` | RISC-V 32-bit |
| `riscv32imc-unknown-none-elf` | RISC-V 32-bit (no atomics) |

### Adding and Building a Target

Install the target toolchain and build:

```bash
# Install the target
rustup target add thumbv7em-none-eabihf

# Build for the target
cargo build --target thumbv7em-none-eabihf
```

The compiled firmware lands in `target/thumbv7em-none-eabihf/debug/` or `release/`.

### Project Configuration

Create `.cargo/config.toml` to avoid typing the target every time:

```toml
[build]
target = "thumbv7em-none-eabihf"

[target.thumbv7em-none-eabihf]
runner = "probe-rs run --chip STM32F411RETx"

rustflags = [
    "-C", "link-arg=-Tlink.x",
]
```

The `runner` field lets `cargo run` flash and run firmware on real hardware via probe-rs. The `link-arg=-Tlink.x` tells the linker to use the `link.x` script from `cortex-m-rt`.

### Memory Layout (memory.x)

The linker needs to know the memory layout of your specific chip:

```
MEMORY
{
    FLASH : ORIGIN = 0x08000000, LENGTH = 512K
    RAM   : ORIGIN = 0x20000000, LENGTH = 128K
}
```

This file is placed at the project root. The values come from the chip's datasheet. Getting these wrong causes hard-to-debug crashes.

### Release Profile for Embedded

Embedded targets are severely constrained in code size. Configure the release profile for size optimization:

```toml
[profile.dev]
opt-level = "s"       # Optimize for size even in debug

[profile.release]
lto = true             # Link-time optimization — removes dead code
opt-level = "z"        # Maximum size optimization
codegen-units = 1      # Better optimization, slower compile
strip = true           # Strip debug symbols from binary
```

The difference between default and size-optimized can be 10x — a 200 KB binary becomes 20 KB.

## Common Pitfalls
1. **Wrong target triple** — Using `thumbv7em-none-eabihf` (hardware float) on a chip without an FPU causes a hard fault on the first floating-point operation. Check your chip's datasheet.
2. **Missing memory.x** — Without a linker script, the linker uses default addresses that do not match your chip. The firmware will not boot.
3. **Forgetting to install the target** — `cargo build` gives a cryptic error if the target is not installed. Always run `rustup target add` first.

## Best Practices
1. **Set the default target in `.cargo/config.toml`** — This prevents accidentally building for the host.
2. **Use `probe-rs` as your runner** — It handles flashing, running, and debug output in a single `cargo run` command.
3. **Always use LTO and size optimization for release** — Flash memory is precious. Enable `lto = true` and `opt-level = "z"` for production firmware.

## Summary
- Target triples identify the architecture, OS, and ABI: `thumbv7em-none-eabihf` means ARM Thumb, no OS, hardware float.
- Install targets with `rustup target add`, build with `cargo build --target`.
- `.cargo/config.toml` configures default target, runner, and linker flags.
- `memory.x` defines FLASH and RAM regions for your specific chip.
- Use LTO and `opt-level = "z"` for release builds to minimize code size.

## Code Examples

**Complete project configuration for embedded Rust — Cargo.toml with size-optimized profiles, .cargo/config.toml with default target and runner, and memory.x with chip-specific layout**

```rust
// Cargo.toml for an embedded project
// [package]
// name = "my-firmware"
// version = "0.1.0"
// edition = "2024"
//
// [dependencies]
// cortex-m = "0.7"
// cortex-m-rt = "0.7"
// panic-halt = "1.0"
// stm32f4xx-hal = { version = "0.21", features = ["stm32f411"] }
//
// [profile.dev]
// opt-level = "s"
//
// [profile.release]
// lto = true
// opt-level = "z"
// codegen-units = 1
// strip = true

// .cargo/config.toml
// [build]
// target = "thumbv7em-none-eabihf"
//
// [target.thumbv7em-none-eabihf]
// runner = "probe-rs run --chip STM32F411RETx"
// rustflags = ["-C", "link-arg=-Tlink.x"]

// memory.x
// MEMORY
// {
//     FLASH : ORIGIN = 0x08000000, LENGTH = 512K
//     RAM   : ORIGIN = 0x20000000, LENGTH = 128K
// }
```


## Resources

- [Rust Embedded Discovery Book](https://docs.rust-embedded.org/discovery/) — Step-by-step guide to setting up your first embedded Rust project
- [probe-rs](https://probe.rs/) — Modern embedded debugging and flashing toolkit

---

> 📘 *This lesson is part of the [Rust for Embedded Systems](https://stanza.dev/courses/rust-embedded) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*