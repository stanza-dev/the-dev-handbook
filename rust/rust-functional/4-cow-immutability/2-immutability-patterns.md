---
source_course: "rust-functional"
source_lesson: "rust-func-immutability-patterns"
---

# Embracing Immutability

Rust defaults to immutable, encouraging functional patterns.

## Shadowing for Transformation

```rust
// Instead of mutation:
let mut x = 5;
x = x + 1;

// Use shadowing:
let x = 5;
let x = x + 1;  // New binding, not mutation
```

## Builder Pattern

```rust
struct Config {
    host: String,
    port: u16,
}

struct ConfigBuilder {
    host: String,
    port: u16,
}

impl ConfigBuilder {
    fn new() -> Self {
        ConfigBuilder {
            host: "localhost".into(),
            port: 8080,
        }
    }
    
    fn host(mut self, host: impl Into<String>) -> Self {
        self.host = host.into();
        self  // Return self for chaining
    }
    
    fn port(mut self, port: u16) -> Self {
        self.port = port;
        self
    }
    
    fn build(self) -> Config {
        Config {
            host: self.host,
            port: self.port,
        }
    }
}

let config = ConfigBuilder::new()
    .host("example.com")
    .port(443)
    .build();
```

## Transforming Instead of Mutating

```rust
// Mutation:
fn add_one_mut(numbers: &mut Vec<i32>) {
    for n in numbers.iter_mut() {
        *n += 1;
    }
}

// Transformation (functional):
fn add_one(numbers: Vec<i32>) -> Vec<i32> {
    numbers.into_iter().map(|n| n + 1).collect()
}

// Or with references:
fn add_one_ref(numbers: &[i32]) -> Vec<i32> {
    numbers.iter().map(|n| n + 1).collect()
}
```

## Code Examples

**Immutable update patterns**

```rust
// Functional data pipeline
fn process_data(data: &[Record]) -> Summary {
    data.iter()
        .filter(|r| r.is_valid())
        .map(|r| r.normalize())
        .fold(Summary::new(), |acc, r| acc.add(r))
}

// Immutable state updates (like Redux)
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
        let mut items = self.items.clone();
        items.push(item);
        AppState { items, ..self.clone() }
    }
}

let state = AppState { count: 0, items: vec![] };
let state = state.increment();
let state = state.add_item("hello".into());
// Each step creates new state, original untouched
```


---

> 📘 *This lesson is part of the [Functional Rust](https://stanza.dev/courses/rust-functional) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*