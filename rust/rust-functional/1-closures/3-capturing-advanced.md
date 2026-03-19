---
source_course: "rust-functional"
source_lesson: "rust-func-closure-capturing-advanced"
---

# Precise Closure Capturing

## Introduction
Since Rust 2021, closures capture individual fields of structs rather than the entire struct. Rust 1.94 further refines this with more precise capturing around pattern bindings. This lesson explains how modern Rust decides what a closure captures and how to control it.

## Key Concepts
- **Disjoint field capture**: Closures capture only the specific fields they use, not the whole struct.
- **Pattern binding captures**: In Rust 1.94, closures that destructure via patterns capture only the bound fields, not the entire binding.
- **Capture precision**: The compiler analyzes each closure body to determine the minimal set of borrows or moves required.

## Real World Context
Precise capturing matters when you have a struct with both `Copy` and non-`Copy` fields. Before disjoint capture, using one field in a closure would move or borrow the entire struct, blocking access to other fields. Modern Rust lets you use different fields in different closures simultaneously.

## Deep Dive

### Disjoint field capture

Consider a struct with two fields:

```rust
struct Player {
    name: String,
    score: u32,
}

let player = Player {
    name: String::from("Alice"),
    score: 100,
};

// This closure captures only `player.name`, not the whole struct
let greet = || println!("Hello, {}", player.name);

// This still works — `player.score` was not captured
println!("Score: {}", player.score);

greet();
```

The closure borrows `player.name` independently of `player.score`. This would have been an error in Rust 2015/2018 editions.

### Pattern binding capture in Rust 1.94

Rust 1.94 refines how closures interact with pattern bindings:

```rust
struct Coordinate {
    x: f64,
    y: f64,
}

let point = Coordinate { x: 1.0, y: 2.0 };

// Destructure, then capture only what is used
let Coordinate { x, y: _ } = &point;
let use_x = || println!("x = {x}");

// y is never captured, so point.y remains freely accessible
println!("y = {}", point.y);
use_x();
```

The compiler recognizes that the closure only needs `x` from the destructured binding, leaving `point.y` unborrowed.

### When full struct capture still happens

If you call a method on the struct itself (not a field), the entire struct is captured:

```rust
struct Config {
    host: String,
    port: u16,
}

impl Config {
    fn display(&self) -> String {
        format!("{}:{}", self.host, self.port)
    }
}

let config = Config { host: "localhost".into(), port: 8080 };

// Captures all of `config` because display() takes &self
let show = || config.display();

// Cannot use config.host here — entire struct is borrowed
show();
```

The method call borrows `&config` as a whole, not individual fields. To work around this, access the fields directly.

### Forcing a move of the whole struct

Sometimes you intentionally want to move the entire struct. Use `move` and reference the struct by name:

```rust
let player = Player {
    name: String::from("Bob"),
    score: 50,
};

let take_player = move || {
    println!("{}: {}", player.name, player.score);
};

// player is fully moved — cannot use any field
take_player();
```

## Common Pitfalls
1. **Assuming whole-struct capture** — Modern Rust captures fields individually. Do not assume that borrowing one field blocks access to others.
2. **Method calls capture the whole struct** — Calling `self.method()` borrows the entire `self`. Access fields directly if you need finer-grained capture.
3. **Edition differences** — Disjoint capture requires Rust 2021 edition or later. If you are on edition 2018, closures capture the whole variable.

## Best Practices
1. **Access fields directly in closures** — Prefer `config.host` over calling `config.display()` when you want precise captures.
2. **Use `let` bindings to narrow captures** — Pull the specific field into a local variable before the closure if you need more control: `let host = &config.host;`.
3. **Test with the 2021 edition** — Ensure your `Cargo.toml` uses `edition = "2021"` or later to benefit from disjoint capture.

## Summary
- Rust 2021+ closures capture individual struct fields, not entire structs.
- Rust 1.94 further refines capturing around pattern bindings.
- Method calls on `self` still capture the whole struct.
- Use direct field access or local bindings to control capture granularity.
- The `move` keyword moves all referenced captures, but still respects disjoint capture for which fields are referenced.

## Code Examples

**Two closures capturing different fields of the same struct simultaneously — enabled by disjoint field capture in Rust 2021+**

```rust
struct Connection {
    host: String,
    port: u16,
    is_secure: bool,
}

fn demonstrate_disjoint_capture() {
    let conn = Connection {
        host: String::from("api.example.com"),
        port: 443,
        is_secure: true,
    };

    // Closure 1: captures only conn.host
    let log_host = || println!("Host: {}", conn.host);

    // Closure 2: captures only conn.port and conn.is_secure
    let describe = || {
        let protocol = if conn.is_secure { "https" } else { "http" };
        format!("{}://...:{}", protocol, conn.port)
    };

    // Both closures can coexist because they capture different fields
    log_host();
    let desc = describe();
    println!("{desc}");
    // Output:
    // Host: api.example.com
    // https://...:443
}
```


## Resources

- [Rust Edition Guide: Disjoint Capture in Closures](https://doc.rust-lang.org/edition-guide/rust-2021/disjoint-capture-in-closures.html) — Official edition guide explaining how Rust 2021 changed closure capturing behavior

---

> 📘 *This lesson is part of the [Functional Rust](https://stanza.dev/courses/rust-functional) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*