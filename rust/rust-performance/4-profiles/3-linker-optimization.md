---
source_course: "rust-performance"
source_lesson: "rust-perf-linker-optimization"
---

# Linker & Toolchain Improvements

## Introduction

Recent Rust releases have brought significant toolchain improvements that affect build times, profiling accuracy, and runtime performance — often requiring zero code changes.

## Key Concepts

### LLD: Default Linker on x86_64-linux (Rust 1.90)

Since Rust 1.90, LLD (the LLVM linker) is the default linker on `x86_64-unknown-linux-gnu`. LLD is significantly faster than the traditional GNU ld:

- **2-5x faster linking** for typical Rust projects
- **Especially impactful** for large binaries with LTO enabled
- **No configuration needed** — it just works

For other targets, you can opt in:

```toml
# .cargo/config.toml
[target.aarch64-unknown-linux-gnu]
linker = "clang"
rustflags = ["-C", "link-arg=-fuse-ld=lld"]
```

### Non-Leaf Frame Pointers on aarch64-linux (Rust 1.89)

Since Rust 1.89, non-leaf frame pointers are enabled by default on `aarch64-unknown-linux-gnu`. This means profilers like perf and samply produce accurate stack traces on ARM Linux without any extra flags.

### Unwind Tables by Default (Rust 1.92)

Even with `-Cpanic=abort`, Rust now emits unwind tables on Linux by default. This gives you proper backtraces in crash reports without the runtime cost of full unwinding.

## Real World Context

These changes particularly benefit CI/CD pipelines (faster linking), ARM server deployments (better profiling), and production debugging (backtraces with abort).

## Deep Dive

### Const Float Operations (Rust 1.90)

`f32` and `f64` methods `floor()`, `ceil()`, `trunc()`, and `round()` are now const-stable. This enables compile-time float computation:

```rust
const GRID_SIZE: f32 = 1.5;
const GRID_CELLS: u32 = GRID_SIZE.ceil() as u32; // Computed at compile time!
```

### Cargo `--timings` (Stable)

The old nightly `-Z timings` flag has been replaced by the stable `--timings` flag:

```bash
cargo build --release --timings
```

This generates an HTML report showing per-crate compile times, parallelism utilization, and the critical path.

### Measuring Link Time

```bash
# Time just the linking step
cargo build --release 2>&1 | grep Linking

# Or use cargo --timings for a visual breakdown
cargo build --release --timings
```

## Common Pitfalls

- Manually configuring LLD on x86_64-linux with Rust 1.90+ — it is already the default.
- Using `-Z timings` with stable Cargo — use `--timings` instead.
- Disabling unwind tables to save binary size without realizing you lose crash backtraces.

## Best Practices

- Let the defaults work for you — Rust 1.90+ has fast linking by default on x86_64-linux.
- Use `--timings` in CI to monitor compilation bottlenecks.
- Use const float operations to move computation from runtime to compile time.
- On aarch64-linux, enjoy accurate profiling out of the box with Rust 1.89+.

## Summary

Recent Rust releases have improved the toolchain significantly: LLD default linker (1.90) for faster builds, default frame pointers on ARM (1.89) for better profiling, unwind tables with abort (1.92) for better debugging, const float ops (1.90) for compile-time computation, and stable `--timings` for build analysis.

## Code Examples

**Const float operations for compile-time computation (Rust 1.90+)**

```rust
// Const float operations (stable since Rust 1.90)
const SCREEN_WIDTH: f32 = 1920.0;
const TILE_SIZE: f32 = 64.0;
const NUM_TILES: u32 = (SCREEN_WIDTH / TILE_SIZE).ceil() as u32;

// Previously required a runtime calculation or manual const
// Now computed at compile time!

fn main() {
    // NUM_TILES is a compile-time constant
    let tiles: Vec<Tile> = Vec::with_capacity(NUM_TILES as usize);
    println!("Tile count: {NUM_TILES}"); // 30
}

struct Tile { x: u32, y: u32 }
```


## Resources

- [Rust 1.90 Release Notes](https://blog.rust-lang.org/2025/09/18/Rust-1.90.0/) — LLD default linker and const float operations
- [Cargo Build Timings](https://doc.rust-lang.org/cargo/reference/timings.html) — Visualize and analyze Cargo build times

---

> 📘 *This lesson is part of the [Rust Performance](https://stanza.dev/courses/rust-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*