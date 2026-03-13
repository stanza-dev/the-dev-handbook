---
source_course: "rust-performance"
source_lesson: "rust-perf-panic-binary"
---

# Panic Strategies & Binary Size

## Introduction

The choice between `panic = "unwind"` and `panic = "abort"` affects binary size, performance, and error handling capabilities. Combined with other settings, you can reduce Rust binaries from megabytes to kilobytes.

## Key Concepts

### Unwind (default)

- Runs destructors (Drop) when panicking
- Supports `catch_unwind` for panic recovery
- Larger binary due to unwinding tables
- Required for libraries (let the caller decide)

### Abort

- Terminates immediately on panic
- No destructors run
- Smaller binary
- Better for applications, embedded, WASM

```toml
[profile.release]
panic = "abort"
```

## Real World Context

Most production applications use `panic = "abort"` since panics indicate bugs that should crash the process. Libraries use unwind to give consumers flexibility. Embedded and WASM always use abort.

## Deep Dive

### Unwind Tables by Default (Rust 1.92)

Since Rust 1.92, unwind tables are emitted by default even with `-Cpanic=abort` on Linux. This means you get proper backtraces in crash reports even in abort mode — previously you would only see a bare abort with no stack trace.

To opt out (for minimal binary size):

```toml
[profile.release]
panic = "abort"
# Add to rustflags to disable unwind tables:
# RUSTFLAGS="-C force-unwind-tables=no"
```

### Binary Size Reduction Checklist

```toml
[profile.release]
panic = "abort"      # Remove unwinding code
strip = true         # Remove symbol table
opt-level = "z"      # Optimize for size
lto = true           # Dead code elimination
codegen-units = 1    # Better DCE
```

### Analyzing Binary Size

```bash
cargo install cargo-bloat
cargo bloat --release --crates   # Size by crate
cargo bloat --release -n 20      # Top 20 functions by size
```

### no_std for Minimal Binaries

```rust
#![no_std]
#![no_main]

#[panic_handler]
fn panic(_: &core::panic::PanicInfo) -> ! {
    loop {}
}
```

A no_std binary can be under 10 KB.

## Common Pitfalls

- Using `panic = "abort"` in a library crate — this forces all consumers to abort on panic.
- Forgetting that `catch_unwind` does not work with `panic = "abort"`.
- Expecting `strip = true` to remove DWARF debug info — you also need to set `debug = false`.

## Best Practices

- Use `panic = "abort"` for applications and `"unwind"` for libraries.
- Use `cargo-bloat` to find what is taking space before blindly applying settings.
- Combine `lto + codegen-units = 1 + strip` for maximum dead-code elimination.
- On Rust 1.92+, enjoy proper backtraces even with abort — only disable unwind tables if binary size is critical.

## Summary

Panic strategy choice affects binary size and error recovery. Abort is smaller and simpler for applications. Since Rust 1.92, you get backtraces even with abort on Linux. Combine panic = abort, strip, LTO, and size optimization for the smallest binaries.

## Code Examples

**Using catch_unwind for panic recovery (requires unwind strategy)**

```rust
// catch_unwind: recovers from panics (requires panic="unwind")
use std::panic;

fn risky_operation() {
    panic!("something went wrong");
}

fn main() {
    let result = panic::catch_unwind(|| {
        risky_operation();
    });

    match result {
        Ok(()) => println!("No panic"),
        Err(_) => println!("Caught a panic!"),
    }
    println!("Program continues...");

    // Use cases for catch_unwind:
    // - FFI boundaries (don't unwind into C)
    // - Thread pools (don't crash the entire pool)
    // - Plugin/WASM sandboxing
}
```


## Resources

- [Rust 1.92 Release Notes](https://blog.rust-lang.org/2025/12/11/Rust-1.92.0/) — Unwind tables enabled by default even with panic=abort
- [cargo-bloat](https://github.com/RazrFalcon/cargo-bloat) — Find what takes most space in your Rust executable

---

> 📘 *This lesson is part of the [Rust Performance](https://stanza.dev/courses/rust-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*