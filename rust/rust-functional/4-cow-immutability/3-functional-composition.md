---
source_course: "rust-functional"
source_lesson: "rust-func-functional-composition"
---

# Functional Composition Patterns

## Introduction
Functional programming is about composing small, focused functions into larger behaviors. While Rust does not have built-in function composition operators like Haskell's `.` or F#'s `>>`, it provides closures, trait objects, and generic functions that enable powerful composition patterns. This lesson covers practical techniques for composing functions in Rust.

## Key Concepts
- **Function composition**: Combining two functions `f` and `g` into a new function `h` where `h(x) = g(f(x))`.
- **Higher-order functions**: Functions that accept or return other functions. `map`, `filter`, and `fold` are all higher-order functions.
- **Pipe pattern**: Passing data through a sequence of transformations, similar to Unix pipes.

## Real World Context
Composition is how middleware stacks work in web frameworks (Actix, Axum), how validation chains operate, and how data pipelines are built. When you chain `.map().filter().fold()`, you are composing functions.

## Deep Dive

### Manual function composition

You can compose two functions using a closure:

```rust
fn compose<A, B, C>(
    f: impl Fn(A) -> B,
    g: impl Fn(B) -> C,
) -> impl Fn(A) -> C {
    move |x| g(f(x))
}

let double = |x: i32| x * 2;
let add_one = |x: i32| x + 1;

let double_then_add = compose(double, add_one);
assert_eq!(double_then_add(5), 11); // 5*2=10, 10+1=11
```

The `compose` function returns a new closure that applies `f` first, then `g`.

### Pipe pattern with method chaining

Rust's iterator chains are essentially a pipe pattern:

```rust
let result = vec![1, 2, 3, 4, 5]
    .into_iter()
    .map(|x| x * 2)        // [2, 4, 6, 8, 10]
    .filter(|&x| x > 4)    // [6, 8, 10]
    .map(|x| x.to_string()) // ["6", "8", "10"]
    .collect::<Vec<_>>();
```

Each step takes the output of the previous step as input, forming a pipeline.

### Composing with fold

`fold` is the most powerful iterator consumer — many other methods can be expressed in terms of it:

```rust
let transactions = vec![100.0, -50.0, 200.0, -75.0, 150.0];

// fold composes an accumulation function across all elements
let (total, count) = transactions.iter()
    .fold((0.0_f64, 0_u32), |(sum, count), &amount| {
        (sum + amount, count + 1)
    });

let average = total / count as f64;
println!("Total: {total}, Count: {count}, Average: {average}");
// Output: Total: 325, Count: 5, Average: 65
```

`fold` carries state (the accumulator) through each step — it is function composition with state.

### Trait objects for dynamic dispatch

When you need to compose functions at runtime (unknown at compile time), use trait objects:

```rust
type Transform = Box<dyn Fn(String) -> String>;

fn build_pipeline(steps: Vec<Transform>) -> impl Fn(String) -> String {
    move |input| {
        steps.iter().fold(input, |acc, step| step(acc))
    }
}

let pipeline = build_pipeline(vec![
    Box::new(|s| s.trim().to_string()),
    Box::new(|s| s.to_lowercase()),
    Box::new(|s| s.replace(" ", "_")),
]);

assert_eq!(pipeline("  Hello World  ".into()), "hello_world");
```

This pattern is useful for plugin systems and configurable data transformations.

## Common Pitfalls
1. **Over-abstracting composition** — Rust's type system makes deeply generic composition verbose. Use concrete types when the abstraction does not pay for itself.
2. **Forgetting `move` in returned closures** — Composed closures that capture variables must use `move` if returned from a function.
3. **Performance with `Box<dyn Fn>`** — Dynamic dispatch adds indirection. Use generics for performance-critical paths.

## Best Practices
1. **Use iterator chains as your primary composition tool** — They are idiomatic, zero-cost, and well-understood by all Rust developers.
2. **Reserve trait objects for runtime-configured pipelines** — When the number or type of steps varies at runtime, `Box<dyn Fn>` is appropriate.
3. **Keep composed functions small** — Each function in a composition chain should do one thing. This makes the pipeline readable and testable.

## Summary
- Function composition combines small functions into larger behaviors.
- Iterator chains (map, filter, fold) are Rust's primary composition tool.
- `fold` is the most powerful consumer — it carries state across the entire iteration.
- `Box<dyn Fn>` enables runtime-configurable pipelines.
- Keep each step focused on a single transformation for readability.

## Code Examples

**A composable validation pipeline using Box<dyn Fn> and try_fold — each validator is a pluggable step**

```rust
// A composable validation pipeline
type ValidationResult = Result<String, String>;
type Validator = Box<dyn Fn(&str) -> ValidationResult>;

fn not_empty() -> Validator {
    Box::new(|input| {
        if input.is_empty() {
            Err("Input cannot be empty".into())
        } else {
            Ok(input.to_string())
        }
    })
}

fn max_length(max: usize) -> Validator {
    Box::new(move |input| {
        if input.len() > max {
            Err(format!("Input exceeds {max} characters"))
        } else {
            Ok(input.to_string())
        }
    })
}

fn validate(input: &str, validators: &[Validator]) -> ValidationResult {
    validators.iter().try_fold(input.to_string(), |acc, v| v(&acc))
}

let rules = vec![not_empty(), max_length(50)];
assert!(validate("hello", &rules).is_ok());
assert!(validate("", &rules).is_err());
```


## Resources

- [Closures: Anonymous Functions that Capture Their Environment](https://doc.rust-lang.org/book/ch13-01-closures.html) — Rust Book chapter on closures, which form the foundation of functional composition in Rust

---

> 📘 *This lesson is part of the [Functional Rust](https://stanza.dev/courses/rust-functional) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*