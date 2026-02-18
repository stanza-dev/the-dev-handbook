---
source_course: "rust-performance"
source_lesson: "rust-perf-cargo-profiles"
---

# Cargo.toml Profile Configuration

```toml
[profile.release]
opt-level = 3          # Optimization level (0-3, s, z)
lto = true              # Link-Time Optimization
codegen-units = 1       # Better optimization, slower compile
panic = "abort"         # Smaller binary, no unwinding
strip = true            # Strip symbols

[profile.release-with-debug]
inherits = "release"
debug = true            # Keep debug symbols
strip = false
```

## opt-level

| Level | Effect |
|-------|--------|
| 0 | No optimization (debug default) |
| 1 | Basic optimization |
| 2 | Most optimizations |
| 3 | All optimizations (release default) |
| s | Optimize for size |
| z | Aggressively optimize for size |

## Link-Time Optimization (LTO)

```toml
[profile.release]
# "thin" - faster compile, good optimization
# "fat"/true - slowest compile, best optimization
# false - no LTO
lto = "thin"
```

## codegen-units

```toml
[profile.release]
# 1 = best optimization, slowest compile
# 16 = default release, parallel compilation
# Higher = faster compile, less optimization
codegen-units = 1
```

## Target CPU

```toml
# .cargo/config.toml
[build]
rustflags = ["-C", "target-cpu=native"]

# Or specific CPU
rustflags = ["-C", "target-cpu=skylake"]
```

See [Cargo Profiles](https://doc.rust-lang.org/cargo/reference/profiles.html).

## Code Examples

**Various profiles**

```rust
// Maximum performance Cargo.toml
[profile.release]
opt-level = 3
lto = "fat"
codegen-units = 1
panic = "abort"
strip = true

// Minimum binary size
[profile.min-size]
inherits = "release"
opt-level = "z"
lto = true
codegen-units = 1
panic = "abort"
strip = true

// Fast compile for development
[profile.dev]
opt-level = 0

[profile.dev.package."*"]
# Optimize dependencies even in dev
opt-level = 2

// Build with:
// cargo build --profile min-size
```


## Resources

- [Cargo Profiles](https://doc.rust-lang.org/cargo/reference/profiles.html) — Official Cargo profile docs

---

> 📘 *This lesson is part of the [Rust Performance](https://stanza.dev/courses/rust-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*