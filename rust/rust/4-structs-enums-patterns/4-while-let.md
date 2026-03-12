---
source_course: "rust"
source_lesson: "rust-if-let-while-let"
---

# if let and while let

## Introduction

Full `match` expressions are powerful but verbose when you only care about one specific pattern. The `if let` and `while let` constructs give you concise syntax for single-pattern matching. Rust 1.65 also introduced `let else` for the inverse case. These tools make your code more readable when exhaustive matching is overkill.

## Key Concepts

- **`if let`**: Matches a value against one pattern and runs a block if it matches. Optionally pairs with `else` for the non-matching case.
- **`while let`**: Loops as long as a pattern continues to match. Commonly used with iterators and stack-like structures.
- **`let else`**: Binds a value if the pattern matches, or diverges (returns, panics, breaks) if it does not. Useful for early returns.
- **Let chains** (2024 edition): Combine multiple `let` patterns with `&&` in `if` and `while` conditions, e.g., `if let Some(x) = a && let Some(y) = b { ... }`.

## Real World Context

In production code, you often need to check whether an `Option` is `Some` or a `Result` is `Ok` without caring about the other variants. Using a full `match` for this is unnecessarily verbose. `if let` is idiomatic Rust for extracting values from single-variant checks. `while let` appears frequently when draining collections or processing streams.

## Deep Dive

Instead of a verbose `match` that ignores most arms, use `if let`:

```rust
// Verbose match
match config_max {
    Some(max) => println!("Max is {max}"),
    _ => (),
}

// Concise if let
if let Some(max) = config_max {
    println!("Max is {max}");
}
```

You can add an `else` branch for the non-matching case:

```rust
if let Some(max) = config_max {
    println!("Max is {max}");
} else {
    println!("No max configured");
}
```

`while let` loops until the pattern stops matching:

```rust
let mut stack = vec![1, 2, 3];
while let Some(top) = stack.pop() {
    println!("{top}");
}
// Prints: 3, 2, 1
```

The `let else` construct (Rust 1.65+) handles the "get the value or bail out" pattern:

```rust
fn parse_header(input: &str) -> Option<(&str, &str)> {
    let Some(colon_pos) = input.find(':') else {
        return None; // Must diverge: return, panic, break, continue
    };
    let key = &input[..colon_pos];
    let value = &input[colon_pos + 1..];
    Some((key.trim(), value.trim()))
}
```

This is much cleaner than nesting the logic inside a `match` or `if let`.

### Let Chains (2024 Edition)

Rust 1.88 introduced let chains, allowing you to combine multiple `let` patterns with `&&` inside `if` and `while` conditions:

```rust
fn process(a: Option<i32>, b: Option<i32>) {
    if let Some(x) = a && let Some(y) = b {
        println!("Both present: {x}, {y}");
    }
}
```

You can freely mix boolean expressions with `let` patterns:

```rust
if let Some(x) = opt && x > 0 && let Ok(y) = result {
    println!("x={x}, y={y}");
}
```

Let chains are exclusive to the 2024 edition because they depend on a change to temporary scoping: in the 2024 edition, temporaries created in `if let` conditions are dropped before the `else` block runs. This ensures consistent and predictable drop ordering when chaining multiple patterns.

## Common Pitfalls

1. **Using `if let` when you need exhaustive matching** — `if let` silently ignores non-matching variants. If you need to handle every variant (and want the compiler to enforce it), use `match` instead.
2. **Forgetting that `let else` must diverge** — The `else` block in `let else` must end with a diverging expression (`return`, `panic!`, `break`, `continue`). You cannot simply assign a default value.

## Best Practices

1. **Use `if let` for single-variant checks on `Option` and `Result`** — It is more idiomatic and readable than a `match` with a `_ => ()` arm.
2. **Prefer `let else` for guard clauses** — When a function needs to early-return on failure, `let else` keeps the happy path unindented and readable.

## Summary

- `if let` matches a single pattern concisely; use it when you only care about one variant.
- `while let` loops while a pattern matches, ideal for draining stacks or iterators.
- `let else` (Rust 1.65+) binds a value or diverges, perfect for guard clauses and early returns.
- Use `match` when you need exhaustive handling of all variants.
- Let chains (2024 edition) allow combining multiple `let` patterns with `&&` in conditions.
- These constructs reduce nesting and make common patterns more readable.

## Code Examples

**if let patterns**

```rust
// Processing Option chains
let numbers = vec![Some(1), None, Some(3)];

for num in numbers {
    if let Some(n) = num {
        println!("Found: {n}");
    }
}

// With Result
if let Ok(value) = "42".parse::<i32>() {
    println!("Parsed: {value}");
}
```


## Resources

- [Concise Control Flow with if let](https://doc.rust-lang.org/stable/book/ch06-03-if-let.html) — Official guide to if let syntax
- [Let-else Statements](https://doc.rust-lang.org/rust-by-example/flow_control/let_else.html) — Rust By Example guide to let-else for early returns

---

> 📘 *This lesson is part of the [Rust Essentials](https://stanza.dev/courses/rust) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*