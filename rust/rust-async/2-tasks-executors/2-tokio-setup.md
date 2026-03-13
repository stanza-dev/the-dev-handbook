---
source_course: "rust-async"
source_lesson: "rust-async-tokio-setup"
---

# Tokio Runtime Setup

## Introduction

Tokio is the most widely used async runtime for Rust. It provides a multi-threaded work-stealing executor, async I/O, timers, and synchronization primitives. Setup starts with the `#[tokio::main]` macro.

## Key Concepts

**Quick start:**

```toml
[dependencies]
tokio = { version = "1", features = ["full"] }
```

```rust
#[tokio::main]
async fn main() {
    println!("Hello from Tokio!");
}
```

This expands to:

```rust
fn main() {
    tokio::runtime::Runtime::new()
        .unwrap()
        .block_on(async { println!("Hello from Tokio!"); });
}
```

## Real World Context

Most production Rust services use `#[tokio::main]` at the entry point. Feature gates let you pull in only what you need for smaller binaries.

## Deep Dive

**Runtime flavors:**

```rust
// Multi-threaded (default) — work-stealing scheduler
#[tokio::main]
async fn main() { }

// Single-threaded — no Send requirement, simpler
#[tokio::main(flavor = "current_thread")]
async fn main() { }

// Custom worker count
#[tokio::main(worker_threads = 4)]
async fn main() { }
```

**Manual Runtime::Builder** for full control:

```rust
let rt = tokio::runtime::Builder::new_multi_thread()
    .worker_threads(4)
    .thread_name("my-worker")
    .enable_all()
    .build()?;

rt.block_on(async {
    // Your async code here
});
```

**Feature gates** — don't use `"full"` in production if you want minimal binaries:

```toml
tokio = { version = "1", features = ["rt", "macros", "net", "time"] }
```

## Common Pitfalls

- Using `"full"` features in a library crate — let consumers choose
- Forgetting `enable_all()` on manual builders — I/O and timers won't work
- Using `current_thread` when spawned tasks need `Send` across threads

## Best Practices

- Use `#[tokio::main]` for applications, manual builder for libraries
- Select only the features you need
- Use `current_thread` for simple tools and tests

## Summary

`#[tokio::main]` is the standard entry point. Choose `multi_thread` (default) or `current_thread`. Use `Runtime::Builder` for advanced configuration. Select minimal feature gates for production.

## Code Examples

**Lightweight runtime setup with feature gates and test macro**

```rust
// Feature-gated setup for smaller binaries
// Cargo.toml: tokio = { version = "1", features = ["rt", "macros"] }

#[tokio::main(flavor = "current_thread")]
async fn main() {
    println!("Lightweight single-threaded runtime!");
}

// For tests: #[tokio::test] works the same way
#[tokio::test]
async fn my_test() {
    assert_eq!(2 + 2, 4);
}
```


## Resources

- [Tokio Tutorial](https://tokio.rs/tokio/tutorial) — Official Tokio getting started guide

---

> 📘 *This lesson is part of the [Asynchronous Rust](https://stanza.dev/courses/rust-async) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*