---
source_course: "rust"
source_lesson: "rust-cargo-setup"
---

# Cargo: Rust's Build System & Package Manager

## Introduction

Cargo is Rust's official build system and package manager, and it handles everything from compiling your code to managing dependencies and running tests. Unlike ecosystems where you juggle multiple tools (make, pip, npm), Cargo provides a single, cohesive workflow that every Rust developer uses from day one.

## Key Concepts

- **Cargo**: The build tool and package manager bundled with every Rust installation. It compiles code, downloads dependencies, runs tests, and generates documentation.
- **Crate**: A Rust package — either a library or a binary. Crates are published to crates.io, Rust's package registry.
- **Cargo.toml**: The project manifest file where you declare metadata, dependencies, and build configuration.
- **Cargo.lock**: An auto-generated file that pins exact dependency versions for reproducible builds.

## Real World Context

Every Rust project uses Cargo. When you join a Rust team, `cargo build` and `cargo test` are the first commands you will run. In CI/CD pipelines, `cargo check` is used for fast feedback (it verifies correctness without producing a binary), and `cargo clippy` catches common mistakes. Understanding Cargo is not optional — it is the foundation of every Rust workflow.

## Deep Dive

The essential Cargo commands you will use daily:

| Command | Description |
|---------|-------------|
| `cargo new project_name` | Create a new project |
| `cargo build` | Compile the project |
| `cargo run` | Build and run |
| `cargo check` | Fast compile check (no binary) |
| `cargo test` | Run tests |
| `cargo doc --open` | Generate and open documentation |
| `cargo clippy` | Run the linter |
| `cargo fmt` | Format code |

When you create a new project, Cargo generates this structure:

```
my_project/
├── Cargo.toml      # Project manifest
├── Cargo.lock      # Locked dependencies
└── src/
    ├── main.rs     # Binary entry point
    └── lib.rs      # Library root (optional)
```

The `Cargo.toml` file is where you configure your project and declare dependencies:

```toml
[package]
name = "my_project"
version = "0.1.0"
edition = "2024"

[dependencies]
serde = { version = "1.0", features = ["derive"] }
tokio = { version = "1", features = ["full"] }

[dev-dependencies]
criterion = "0.5"  # Only for tests and benchmarks
```

The `edition` field specifies which Rust edition to use. Editions (2015, 2018, 2021, 2024) introduce new syntax and features while maintaining backward compatibility.

Cargo supports two build profiles. Debug mode compiles fast but produces slower code. Release mode takes longer to compile but optimizes aggressively:

```bash
cargo build          # Debug (fast compile, slow code)
cargo build --release # Release (slow compile, fast code)
```

## Common Pitfalls

1. **Forgetting `--release` for benchmarks** — Running benchmarks in debug mode gives misleading results because optimizations are disabled. Always use `cargo build --release` for performance testing.
2. **Editing `Cargo.lock` manually** — This file is auto-generated. Let Cargo manage it. Commit it for binaries (reproducible builds), but gitignore it for libraries.

## Best Practices

1. **Use `cargo check` for fast feedback** — It skips code generation and runs 2-3x faster than `cargo build`. IDEs use it for real-time error reporting.
2. **Run `cargo clippy` regularly** — Clippy catches hundreds of common mistakes and suggests idiomatic Rust patterns. Treat its warnings as errors in CI.

## Summary

- Cargo is the single tool for building, testing, formatting, linting, and managing dependencies in Rust.
- `cargo new` scaffolds a project; `cargo run` builds and executes it.
- `cargo check` is the fastest way to verify your code compiles correctly.
- `Cargo.toml` declares your project metadata and dependencies; `Cargo.lock` pins exact versions.
- Use `--release` for production builds and benchmarks.

## Code Examples

**Basic cargo workflow**

```bash
# Create and run a project
cargo new hello_rust
cd hello_rust
cargo run

# Add a dependency
cargo add serde --features derive
```


## Resources

- [The Cargo Book](https://doc.rust-lang.org/cargo/) — Complete guide to Cargo

---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*