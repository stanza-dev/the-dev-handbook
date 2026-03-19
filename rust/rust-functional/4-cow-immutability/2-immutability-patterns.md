---
source_course: "rust-functional"
source_lesson: "rust-func-immutability-patterns"
---

# Immutability Patterns

## Introduction
Rust defaults to immutable bindings, encouraging a functional style where data is transformed rather than mutated in place. This lesson covers practical patterns for working with immutable data: shadowing, the builder pattern, and transformation pipelines.

## Key Concepts
- **Shadowing**: Rebinding a variable name with `let` to a new value, rather than mutating the original.
- **Builder pattern**: A struct with chainable methods that produce a final, immutable configuration object.
- **Transformation over mutation**: Creating new collections via iterator pipelines instead of modifying existing ones.

## Real World Context
Immutability makes code easier to reason about, especially in concurrent programs where shared mutable state is the root of most bugs. Rust's ownership system makes immutable patterns zero-cost — there is no garbage collector overhead for creating new values.

## Deep Dive

### Shadowing for step-by-step transformation

Instead of mutating a variable, create new bindings with the same name:

```rust
let input = "  42  ";
let input = input.trim();        // &str, whitespace removed
let input: i32 = input.parse().unwrap(); // i32, parsed
let input = input * 2;           // i32, doubled
assert_eq!(input, 84);
```

Each `let` creates a new binding. The previous value is dropped (or the borrow ends). This is not mutation — each step can even change the type.

### The builder pattern

Builders let you configure complex objects step by step, producing an immutable result:

```rust
struct ServerConfig {
    host: String,
    port: u16,
    max_connections: usize,
}

struct ServerConfigBuilder {
    host: String,
    port: u16,
    max_connections: usize,
}

impl ServerConfigBuilder {
    fn new() -> Self {
        ServerConfigBuilder {
            host: "127.0.0.1".into(),
            port: 8080,
            max_connections: 100,
        }
    }

    fn host(mut self, host: impl Into<String>) -> Self {
        self.host = host.into();
        self
    }

    fn port(mut self, port: u16) -> Self {
        self.port = port;
        self
    }

    fn max_connections(mut self, max: usize) -> Self {
        self.max_connections = max;
        self
    }

    fn build(self) -> ServerConfig {
        ServerConfig {
            host: self.host,
            port: self.port,
            max_connections: self.max_connections,
        }
    }
}
```

The builder consumes `self` at each step (taking ownership), so the chain is type-safe. The final `build()` returns an immutable `ServerConfig`:

```rust
let config = ServerConfigBuilder::new()
    .host("0.0.0.0")
    .port(443)
    .max_connections(10_000)
    .build();
```

### Transformation over mutation

Instead of modifying a collection in place, create a new one:

```rust
// Mutation style (imperative)
fn add_tax_mut(prices: &mut Vec<f64>, rate: f64) {
    for price in prices.iter_mut() {
        *price *= 1.0 + rate;
    }
}

// Transformation style (functional)
fn add_tax(prices: &[f64], rate: f64) -> Vec<f64> {
    prices.iter().map(|price| price * (1.0 + rate)).collect()
}
```

The transformation style is easier to test (no side effects), compose, and parallelize. The original data remains unchanged for other uses.

### Immutable state updates

For application state, create new versions rather than mutating:

```rust
#[derive(Clone)]
struct AppState {
    count: i32,
    items: Vec<String>,
}

impl AppState {
    fn increment(&self) -> Self {
        AppState {
            count: self.count + 1,
            ..self.clone()
        }
    }

    fn add_item(&self, item: String) -> Self {
        let mut new_items = self.items.clone();
        new_items.push(item);
        AppState {
            items: new_items,
            ..self.clone()
        }
    }
}
```

Each method returns a new state. The original is untouched, enabling easy undo/redo and state history.

## Common Pitfalls
1. **Excessive cloning for immutability** — Cloning large data structures just to avoid mutation can be expensive. Use `Cow`, `Rc`, or `Arc` for shared ownership instead.
2. **Confusing shadowing with mutation** — Shadowing creates a new binding. The old value may still exist if references to it remain.
3. **Builder without consuming `self`** — If builder methods take `&mut self` instead of `self`, users can accidentally reuse a partially-built builder.

## Best Practices
1. **Use shadowing for sequential transformations** — It makes the data flow explicit and avoids naming intermediate values.
2. **Consume `self` in builder methods** — This prevents misuse and makes the builder pattern type-safe.
3. **Combine immutability with iterators** — Iterator chains naturally produce new data without mutation.

## Summary
- Rust defaults to immutable bindings, encouraging functional data flow.
- Shadowing replaces mutation for step-by-step transformations.
- The builder pattern produces immutable objects from a chainable API.
- Transformation pipelines create new collections rather than modifying existing ones.
- Use `Cow`, `Rc`, or `Arc` to avoid expensive clones for shared data.

## Code Examples

**An immutable sales data pipeline using fold and iterator transformations — the original data is never modified**

```rust
// Immutable data processing pipeline
struct SalesRecord {
    product: String,
    amount: f64,
    region: String,
}

fn summarize_by_region(records: &[SalesRecord]) -> Vec<(String, f64)> {
    use std::collections::HashMap;

    let totals: HashMap<&str, f64> = records.iter()
        .fold(HashMap::new(), |mut acc, record| {
            *acc.entry(&record.region).or_insert(0.0) += record.amount;
            acc
        });

    // Transform into sorted Vec without mutating the HashMap
    let mut result: Vec<(String, f64)> = totals
        .into_iter()
        .map(|(region, total)| (region.to_string(), total))
        .collect();

    result.sort_by(|a, b| b.1.partial_cmp(&a.1).unwrap());
    result
}
// Original records remain unchanged — pure transformation
```


## Resources

- [Variables and Mutability](https://doc.rust-lang.org/book/ch03-01-variables-and-mutability.html) — Rust Book chapter on variables, mutability, and shadowing

---

> 📘 *This lesson is part of the [Functional Rust](https://stanza.dev/courses/rust-functional) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*