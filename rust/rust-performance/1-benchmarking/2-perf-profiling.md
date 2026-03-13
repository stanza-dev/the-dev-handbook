---
source_course: "rust-performance"
source_lesson: "rust-perf-profiling"
---

# Profiling Tools

## Introduction

Benchmarks tell you how fast something is. Profilers tell you why it is slow. A complete performance workflow combines both: benchmark to quantify, profile to diagnose.

## Key Concepts

### CPU Profiling

**perf (Linux):**

```bash
cargo build --release
perf record -g ./target/release/myapp
perf report
```

**samply (Cross-platform):**

samply is a modern sampling profiler that outputs Firefox Profiler format. It works on Linux, macOS, and Windows:

```bash
cargo install samply
samply record ./target/release/myapp
# Opens Firefox Profiler in your browser
```

**Flamegraph:**

```bash
cargo install flamegraph
cargo flamegraph --release
```

**Instruments (macOS):**

```bash
cargo build --release
open -a Instruments ./target/release/myapp
```

### Memory Profiling

**DHAT (Heap Profiler):**

```rust
[dependencies]
dhat = { version = "0.3", optional = true }

[features]
dhat-heap = ["dhat"]
```

```rust
#[cfg(feature = "dhat-heap")]
#[global_allocator]
static ALLOC: dhat::Alloc = dhat::Alloc;

fn main() {
    #[cfg(feature = "dhat-heap")]
    let _profiler = dhat::Profiler::new_heap();
    // Your code here
}
```

**Valgrind (Linux):**

```bash
valgrind --tool=massif ./target/release/myapp
ms_print massif.out.*
```

## Real World Context

Production teams use samply or perf to find hot functions, DHAT to find excessive allocations, and flamegraphs to visualize the call stack in CI dashboards.

## Deep Dive

### Compile-Time Profiling

Cargo now provides stable `--timings` (the old `-Z timings` flag is gone):

```bash
cargo build --release --timings
# Opens an HTML report showing per-crate compile times
```

For self-profiling of rustc itself (nightly):

```bash
RUSTFLAGS="-Z self-profile" cargo build --release
```

### Non-Leaf Frame Pointers on aarch64-linux

Since Rust 1.89, non-leaf frame pointers are enabled by default on `aarch64-unknown-linux-gnu`. This means profilers like perf and samply produce accurate stack traces out of the box on ARM Linux — no extra flags needed.

### Manual Instrumentation

For targeted profiling, use `std::time::Instant`:

```rust
let start = std::time::Instant::now();
expensive_operation();
eprintln!("Took: {:?}", start.elapsed());
```

For production, prefer the `tracing` crate with timing spans.

## Common Pitfalls

- Profiling debug builds — always profile release builds with debug symbols: `[profile.release] debug = true`.
- Using `-Z timings` with recent Cargo — use `--timings` instead (stable since Cargo 1.60+).
- Forgetting to enable frame pointers on x86_64 for accurate stack traces: add `-C force-frame-pointers=yes`.

## Best Practices

- Use samply for quick cross-platform profiling with a modern UI.
- Keep `debug = true` in your release profile for symbol information.
- Combine CPU and memory profiling — a function may be slow because it allocates too much.
- Use `--timings` in CI to track compile-time regressions.

## Summary

Rust has excellent profiling support across platforms. Use perf or samply for CPU profiling, DHAT or Valgrind for memory profiling, flamegraphs for visualization, and `cargo build --timings` for compile-time analysis. Since Rust 1.89, aarch64-linux gets better profiling out of the box with default frame pointers.

## Code Examples

**Manual timing macro for quick profiling**

```rust
use std::time::Instant;

macro_rules! time_it {
    ($name:expr, $block:expr) => {{
        let start = Instant::now();
        let result = $block;
        let duration = start.elapsed();
        eprintln!("{}: {:?}", $name, duration);
        result
    }};
}

fn main() {
    let data = time_it!("generate", {
        (0..1_000_000).collect::<Vec<i32>>()
    });

    let sorted = time_it!("sort", {
        let mut d = data.clone();
        d.sort_unstable();
        d
    });

    let _sum: i32 = time_it!("sum", {
        sorted.iter().sum()
    });
}
```


## Resources

- [samply — Sampling Profiler](https://github.com/mstange/samply) — Cross-platform sampling profiler with Firefox Profiler output
- [The Rust Performance Book](https://nnethercote.github.io/perf-book/) — Comprehensive Rust performance guide including profiling
- [cargo-flamegraph](https://github.com/flamegraph-rs/flamegraph) — Generate flamegraphs from Rust programs

---

> 📘 *This lesson is part of the [Rust Performance](https://stanza.dev/courses/rust-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*