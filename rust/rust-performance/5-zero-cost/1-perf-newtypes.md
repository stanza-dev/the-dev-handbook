---
source_course: "rust-performance"
source_lesson: "rust-perf-newtypes"
---

# The Newtype Pattern

Wrap a type for added safety with zero runtime cost:

```rust
// Without newtype - easy to mix up
fn connect(host: &str, port: u16, timeout_ms: u64) { }
connect("localhost", 5000, 8080);  // Oops! Swapped port and timeout

// With newtypes - compile-time safety
struct Port(u16);
struct TimeoutMs(u64);

fn connect(host: &str, port: Port, timeout: TimeoutMs) { }
connect("localhost", Port(8080), TimeoutMs(5000));  // Clear!
// connect("localhost", TimeoutMs(5000), Port(8080));  // Compile error!
```

## Zero Cost

```rust
use std::mem::size_of;

struct UserId(u64);

assert_eq!(size_of::<u64>(), size_of::<UserId>());  // Same size!
// No overhead - it's just a u64 with a different type
```

## #[repr(transparent)]

Guarantees same layout as the inner type:

```rust
#[repr(transparent)]
struct Wrapper(u32);

// Safe to transmute between Wrapper and u32
// Useful for FFI
```

## Common Uses

```rust
// Units
struct Meters(f64);
struct Feet(f64);

impl Meters {
    fn to_feet(self) -> Feet {
        Feet(self.0 * 3.28084)
    }
}

// IDs that shouldn't mix
struct UserId(u64);
struct OrderId(u64);

// Validated strings
struct Email(String);

impl Email {
    fn new(s: &str) -> Result<Self, &'static str> {
        if s.contains('@') {
            Ok(Email(s.to_string()))
        } else {
            Err("Invalid email")
        }
    }
}
```

## Code Examples

**Newtype with Deref**

```rust
// Newtype with Deref for convenience
use std::ops::Deref;

struct Name(String);

impl Deref for Name {
    type Target = str;
    
    fn deref(&self) -> &str {
        &self.0
    }
}

let name = Name("Alice".to_string());

// Can use &str methods directly
println!("Length: {}", name.len());
println!("Upper: {}", name.to_uppercase());

// But still type-safe
fn greet(name: &Name) {
    println!("Hello, {name}!");
}

greet(&name);  // Works
// greet(&"Bob".to_string());  // Error: expected &Name
```


---

> 📘 *This lesson is part of the [Rust Performance](https://stanza.dev/courses/rust-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*