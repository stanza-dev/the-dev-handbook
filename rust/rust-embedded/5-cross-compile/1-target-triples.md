---
source_course: "rust-embedded"
source_lesson: "rust-embedded-target-triples"
---

# Cross-Compilation

Build for a different architecture than your host.

## Target Triple Format

```
<arch>-<vendor>-<os>-<abi>
```

## Common Embedded Targets

| Target | Description |
|--------|-------------|
| `thumbv7em-none-eabihf` | Cortex-M4F with FPU |
| `thumbv7em-none-eabi` | Cortex-M4 no FPU |
| `thumbv7m-none-eabi` | Cortex-M3 |
| `thumbv6m-none-eabi` | Cortex-M0/M0+ |
| `riscv32imac-unknown-none-elf` | RISC-V 32-bit |
| `wasm32-unknown-unknown` | WebAssembly |

## Adding a Target

```bash
rustup target add thumbv7em-none-eabihf
```

## Building for Target

```bash
cargo build --target thumbv7em-none-eabihf
```

## .cargo/config.toml

```toml
[build]
target = "thumbv7em-none-eabihf"

[target.thumbv7em-none-eabihf]
runner = "probe-run --chip STM32F411RE"

rustflags = [
    "-C", "link-arg=-Tlink.x",
]
```

## Memory Layout (memory.x)

```
MEMORY
{
    FLASH : ORIGIN = 0x08000000, LENGTH = 512K
    RAM : ORIGIN = 0x20000000, LENGTH = 128K
}
```

## Code Examples

**Project configuration**

```rust
// Example Cargo.toml for embedded
[package]
name = "my-embedded-app"
version = "0.1.0"
edition = "2021"

[dependencies]
cortex-m = "0.7"
cortex-m-rt = "0.7"
panic-halt = "0.2"
stm32f4xx-hal = { version = "0.17", features = ["stm32f411"] }

[profile.dev]
opt-level = "s"  # Optimize for size even in debug

[profile.release]
lto = true
opt-level = "z"  # Maximum size optimization
codegen-units = 1

// Example .cargo/config.toml
[target.thumbv7em-none-eabihf]
runner = "probe-run --chip STM32F411RE"

[target.'cfg(all(target_arch = "arm", target_os = "none"))']
rustflags = [
    "-C", "link-arg=-Tlink.x",
    "-C", "link-arg=--nmagic",
]

[build]
target = "thumbv7em-none-eabihf"
```


---

> 📘 *This lesson is part of the [Rust for Embedded Systems](https://stanza.dev/courses/rust-embedded) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*