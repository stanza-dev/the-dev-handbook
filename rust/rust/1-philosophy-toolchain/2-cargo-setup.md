---
source_course: "rust"
source_lesson: "rust-cargo-setup"
---

# Cargo: Your Best Friend

Cargo handles building, testing, documentation, and dependency management.

## Essential Commands

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

## Project Structure

```
my_project/
├── Cargo.toml      # Project manifest
├── Cargo.lock      # Locked dependencies
└── src/
    ├── main.rs     # Binary entry point
    └── lib.rs      # Library root (optional)
```

## Cargo.toml Anatomy

```toml
[package]
name = "my_project"
version = "0.1.0"
edition = "2021"  # Use the latest edition!

[dependencies]
serde = { version = "1.0", features = ["derive"] }
tokio = { version = "1", features = ["full"] }

[dev-dependencies]
criterion = "0.5"  # Only for tests/benchmarks
```

## Build Profiles

```bash
cargo build          # Debug (fast compile, slow code)
cargo build --release # Release (slow compile, fast code)
```

See [The Cargo Book](https://doc.rust-lang.org/cargo/).

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