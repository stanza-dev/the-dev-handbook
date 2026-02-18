---
source_course: "rust-performance"
source_lesson: "rust-perf-panic-strategies"
---

# Panic Strategies

## panic = "unwind" (default)

- Runs destructors when panicking
- Supports `catch_unwind`
- Larger binary (unwinding tables)
- Slower panics

## panic = "abort"

- Program terminates immediately
- No destructors run
- Smaller binary
- Faster panics (they're rare anyway)

```toml
[profile.release]
panic = "abort"
```

## When to Use Each

| Use Case | Strategy |
|----------|----------|
| Libraries | unwind (let caller decide) |
| Applications | abort (usually fine) |
| Need catch_unwind | unwind |
| Embedded/WASM | abort |
| Minimize binary | abort |

# Binary Size Reduction

```toml
[profile.release]
panic = "abort"         # Remove unwinding
strip = true            # Remove symbols
opt-level = "z"         # Size optimization
lto = true              # Dead code elimination
codegen-units = 1       # Better DCE
```

## More Techniques

```bash
# Use cargo-bloat to find what's taking space
cargo install cargo-bloat
cargo bloat --release

# Strip after build
strip target/release/myapp

# UPX compression (controversial)
upx --best target/release/myapp
```

## std vs no_std

```rust
// Smallest possible binary (no std)
#![no_std]
#![no_main]

#[panic_handler]
fn panic(_: &core::panic::PanicInfo) -> ! {
    loop {}
}
```

## Code Examples

**catch_unwind usage**

```rust
// Using catch_unwind (requires panic="unwind")
use std::panic;

fn might_panic() {
    panic!("oops!");
}

fn main() {
    let result = panic::catch_unwind(|| {
        might_panic();
    });
    
    match result {
        Ok(()) => println!("No panic"),
        Err(_) => println!("Caught a panic!"),
    }
    
    println!("Program continues...");
}

// Note: catch_unwind is NOT for normal error handling!
// Use it for:
// - FFI boundaries (don't let panics cross into C)
// - Thread pools (don't crash the whole pool)
// - Plugin systems
```


---

> 📘 *This lesson is part of the [Rust Performance](https://stanza.dev/courses/rust-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*