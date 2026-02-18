---
source_course: "rust-performance"
source_lesson: "rust-perf-profiling"
---

# Profiling Rust Programs

## CPU Profiling

### perf (Linux)

```bash
# Record
cargo build --release
perf record -g ./target/release/myapp
perf report

# Or with Rust symbols
RUST_BACKTRACE=1 perf record -g ./target/release/myapp
```

### Instruments (macOS)

```bash
cargo build --release
open -a Instruments ./target/release/myapp
```

### Flamegraph

```bash
cargo install flamegraph

# Linux (needs perf)
cargo flamegraph --release

# macOS (needs dtrace, run as root)
sudo cargo flamegraph --release
```

## Memory Profiling

### DHAT (Heap Profiler)

```rust
// Add to Cargo.toml
[dependencies]
dhat = { version = "0.3", optional = true }

[features]
dhat-heap = ["dhat"]

// In main.rs
#[cfg(feature = "dhat-heap")]
#[global_allocator]
static ALLOC: dhat::Alloc = dhat::Alloc;

fn main() {
    #[cfg(feature = "dhat-heap")]
    let _profiler = dhat::Profiler::new_heap();
    
    // Your code here
}
```

### Valgrind (Linux)

```bash
cargo build --release
valgrind --tool=massif ./target/release/myapp
ms_print massif.out.*
```

## Compile Time Profiling

```bash
# Time where compilation spends time
cargo build -Z timings --release

# Self-profiling (nightly)
RUSTFLAGS="-Z self-profile" cargo build --release
```

## Code Examples

**Manual timing macro**

```rust
// Manual instrumentation for profiling
use std::time::Instant;

macro_rules! time_it {
    ($name:expr, $block:block) => {{
        let start = Instant::now();
        let result = $block;
        let duration = start.elapsed();
        println!("{}: {:?}", $name, duration);
        result
    }};
}

fn main() {
    let data = time_it!("generate data", {
        (0..1_000_000).collect::<Vec<_>>()
    });
    
    let sorted = time_it!("sort", {
        let mut d = data.clone();
        d.sort();
        d
    });
    
    let sum: i32 = time_it!("sum", {
        sorted.iter().sum()
    });
    
    println!("Sum: {sum}");
}

// Better: Use tracing crate for production
```


## Resources

- [Rust Performance Book](https://nnethercote.github.io/perf-book/) — Comprehensive Rust performance guide

---

> 📘 *This lesson is part of the [Rust Performance](https://stanza.dev/courses/rust-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*