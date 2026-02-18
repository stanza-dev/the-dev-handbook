---
source_course: "rust-async"
source_lesson: "rust-async-tokio-basics"
---

# The Tokio Runtime

Tokio is the most widely-used async runtime for Rust.

## Quick Setup

```toml
# Cargo.toml
[dependencies]
tokio = { version = "1", features = ["full"] }
```

## The #[tokio::main] Macro

```rust
#[tokio::main]
async fn main() {
    println!("Hello from async!");
}
```

This expands to:

```rust
fn main() {
    tokio::runtime::Runtime::new()
        .unwrap()
        .block_on(async {
            println!("Hello from async!");
        });
}
```

## Runtime Flavors

```rust
// Multi-threaded (default) - best for I/O bound work
#[tokio::main]
async fn main() { }

// Single-threaded - simpler, good for !Send futures
#[tokio::main(flavor = "current_thread")]
async fn main() { }

// Custom worker count
#[tokio::main(worker_threads = 4)]
async fn main() { }
```

## Manual Runtime Configuration

```rust
let rt = tokio::runtime::Builder::new_multi_thread()
    .worker_threads(4)
    .thread_name("my-worker")
    .thread_stack_size(3 * 1024 * 1024)
    .enable_all()
    .build()?;

rt.block_on(async {
    // Your async code
});
```

See [Tokio Tutorial](https://tokio.rs/tokio/tutorial).

## Code Examples

**Feature selection for Tokio**

```rust
// Feature-gated setup for smaller binaries
// Cargo.toml: tokio = { version = "1", features = ["rt", "macros"] }

#[tokio::main(flavor = "current_thread")]
async fn main() {
    // Minimal runtime, single thread
    println!("Lightweight async!");
}

// Or with specific features:
// features = ["rt-multi-thread", "time", "net", "io-util", "macros"]
```


## Resources

- [Tokio Tutorial](https://tokio.rs/tokio/tutorial) — Official Tokio getting started guide

---

> 📘 *This lesson is part of the [Asynchronous Rust](https://stanza.dev/courses/rust-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*