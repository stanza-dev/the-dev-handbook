---
source_course: "rust"
source_lesson: "rust-variables-and-mutability"
---

# Immutable by Default

In Rust, variables are immutable by default. This is a design choice to make code easier to reason about and prevent accidental mutations.

```rust
let x = 5;
// x = 6; // Error! Cannot assign twice to immutable variable
```

To make it mutable, use `mut`:

```rust
let mut y = 10;
y = 15; // OK
y += 1; // Also OK
```

## Constants vs Variables

Constants are different from immutable variables:

```rust
const MAX_POINTS: u32 = 100_000; // MUST have type annotation
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3; // Computed at compile time
```

**Key differences:**
- Constants are ALWAYS immutable (no `mut`)
- Constants require type annotations
- Constants can only be set to constant expressions
- Constants can be declared in any scope, including global

## Shadowing: A Powerful Pattern

You can declare a new variable with the same name as a previous one. This is called **shadowing**.

```rust
let spaces = "   ";        // &str
let spaces = spaces.len(); // usize - type changed!

// vs mutation (same type required)
let mut spaces = "   ";
// spaces = spaces.len(); // Error! Can't change type
```

**When to use shadowing:**
- Transforming a value through several steps
- Converting types while keeping the same name
- Narrowing scope of mutability

```rust
let guess: u32 = "42".parse().expect("Not a number");
// guess is now a u32, not a string
```

See [Variables and Mutability](https://doc.rust-lang.org/stable/book/ch03-01-variables-and-mutability.html).

## Code Examples

**Shadowing in nested scopes**

```rust
fn main() {
    let x = 5;
    let x = x + 1; // Shadow with new value
    {
        let x = x * 2; // Inner scope shadow
        println!("Inner x: {x}"); // 12
    }
    println!("Outer x: {x}"); // 6
}
```


---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*