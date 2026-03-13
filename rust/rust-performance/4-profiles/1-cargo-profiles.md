---
source_course: "rust-performance"
source_lesson: "rust-perf-cargo-profiles"
---

# Cargo Profile Settings

## Introduction

Cargo profiles control how your code is compiled. The difference between a debug and a fully optimized release build can be 10-100x in performance. Understanding each setting lets you find the right tradeoff for your use case.

## Key Concepts

### Core Settings

```toml
[profile.release]
opt-level = 3          # Optimization level (0-3, s, z)
lto = true             # Link-Time Optimization
codegen-units = 1      # Better optimization, slower compile
panic = "abort"        # Smaller binary, no unwinding
strip = true           # Strip debug symbols
```

### opt-level

| Level | Effect | Use Case |
|-------|--------|----------|
| 0 | No optimization | Debug builds |
| 1 | Basic optimizations | Faster debug |
| 2 | Most optimizations | Good balance |
| 3 | All optimizations | Maximum speed (release default) |
| "s" | Optimize for size | Embedded, WASM |
| "z" | Aggressively minimize size | Smallest binary |

### Link-Time Optimization (LTO)

```toml
[profile.release]
lto = "thin"   # Fast compile, good optimization
lto = "fat"    # Slow compile, best optimization (same as true)
lto = false    # No cross-crate optimization
```

LTO enables the compiler to optimize across crate boundaries, inlining functions from dependencies and eliminating dead code globally.

### codegen-units

```toml
[profile.release]
codegen-units = 1   # Best optimization (default: 16)
```

With 1 codegen unit, the compiler sees all code at once, enabling better inlining and optimization. The tradeoff is slower compilation.

## Real World Context

Production Rust services typically use `lto = "thin"` for a good compile-time/runtime tradeoff. Performance-critical CLI tools use `lto = "fat"` with `codegen-units = 1`. WASM binaries use `opt-level = "z"` for download size.

## Deep Dive

### Target CPU

```toml
# .cargo/config.toml
[build]
rustflags = ["-C", "target-cpu=native"]
```

This enables CPU-specific optimizations including SIMD instruction sets. For portable binaries, use a baseline like `x86-64-v3` (AVX2).

### Custom Profiles

```toml
[profile.release-fast]
inherits = "release"
opt-level = 3
lto = "fat"
codegen-units = 1

[profile.release-debug]
inherits = "release"
debug = true
strip = false
```

Build with: `cargo build --profile release-fast`

### Optimizing Dependencies in Dev

```toml
[profile.dev.package."*"]
opt-level = 2  # Optimize deps even in debug mode
```

This keeps your code unoptimized (fast compile, debuggable) while optimizing dependencies (faster runtime for deps like regex, serde).

## Common Pitfalls

- Forgetting to benchmark in release mode — `cargo bench` uses release by default, but `cargo run` uses debug.
- Setting `codegen-units = 1` in CI without caching — it dramatically slows compilation.
- Using `lto = "fat"` when `"thin"` would give 95% of the benefit at 50% of the compile time.

## Best Practices

- Use `lto = "thin"` as a default for production release profiles.
- Use `codegen-units = 1` only for final release builds, not for CI.
- Add `[profile.dev.package."*"] opt-level = 2` to speed up debug builds that depend on heavy crates.
- Create custom profiles for different scenarios (max speed, min size, profiling).

## Summary

Cargo profiles give fine-grained control over compilation. The key settings are opt-level, LTO, codegen-units, and target-cpu. Thin LTO with the default codegen-units is the best starting point; narrow to fat LTO and codegen-units = 1 for final releases.

## Code Examples

**Cargo.toml profiles for maximum performance, minimum size, and fast development**

```bash
# Maximum performance profile
[profile.release]
opt-level = 3
lto = "fat"
codegen-units = 1
panic = "abort"
strip = true

# Minimum binary size
[profile.min-size]
inherits = "release"
opt-level = "z"
lto = true
codegen-units = 1
panic = "abort"
strip = true

# Fast dev builds with optimized dependencies
[profile.dev]
opt-level = 0

[profile.dev.package."*"]
opt-level = 2
```


## Resources

- [Cargo Profiles Reference](https://doc.rust-lang.org/cargo/reference/profiles.html) — Official Cargo documentation on build profiles
- [min-sized-rust](https://github.com/johnthagen/min-sized-rust) — Guide to minimizing Rust binary size

---

> 📘 *This lesson is part of the [Rust Performance](https://stanza.dev/courses/rust-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*