---
source_course: "rust-functional"
source_lesson: "rust-func-iterator-adapters"
---

# Chaining Iterator Adapters

## Introduction
Iterators in Rust are lazy — adapter methods like `map`, `filter`, and `flat_map` build up a pipeline that does no work until a consumer method like `collect` or `sum` drives the iteration. This lesson covers the most important adapters and how to chain them into expressive data pipelines.

## Key Concepts
- **Iterator adapter**: A method on an iterator that returns a new iterator with modified behavior. Adapters are lazy — they build a chain but do not execute it.
- **Consumer (terminal operation)**: A method like `collect()`, `sum()`, `count()`, or `for_each()` that drives the iterator chain to completion.
- **Lazy evaluation**: No element is processed until a consumer requests it. This means an infinite range like `0..` is perfectly safe as long as you limit consumption.

## Real World Context
Iterator chains are the idiomatic way to process collections in Rust. They often compile down to the same machine code as hand-written loops, thanks to the compiler's ability to inline and optimize iterator adapters. If you have used `.map().filter()` in JavaScript, Python, or Java streams, Rust's iterators will feel familiar — but with zero-cost abstraction guarantees.

## Deep Dive

### Core adapters

Here are the adapters you will use most often. Each one takes an iterator and returns a new iterator:

```rust
let temperatures = vec![72.0, 68.5, 75.3, 80.1, 65.0];

// map: transform each element
let celsius: Vec<f64> = temperatures.iter()
    .map(|f| (f - 32.0) * 5.0 / 9.0)
    .collect();
// [22.2, 20.3, 24.1, 26.7, 18.3] (approximate)

// filter: keep elements matching a predicate
let warm: Vec<&f64> = temperatures.iter()
    .filter(|&&temp| temp > 70.0)
    .collect();
// [72.0, 75.3, 80.1]

// filter_map: filter and transform in one step
let parsed: Vec<i32> = ["10", "abc", "30", "xyz"].iter()
    .filter_map(|s| s.parse::<i32>().ok())
    .collect();
// [10, 30]
```

Each adapter returns a new iterator struct. No allocation or computation happens until `collect()` consumes the chain.

### Positioning adapters

These adapters control which elements pass through:

```rust
let numbers = 1..=100;

// take: yield only the first N elements
let first_five: Vec<i32> = numbers.clone().take(5).collect();
// [1, 2, 3, 4, 5]

// skip: discard the first N elements
let after_ten: Vec<i32> = numbers.clone().skip(95).collect();
// [96, 97, 98, 99, 100]

// take_while: yield elements while predicate is true
let small: Vec<i32> = numbers.clone().take_while(|&x| x < 4).collect();
// [1, 2, 3]
```

These are especially useful with infinite iterators — `take(n)` makes them finite.

### Chaining multiple adapters

The real power emerges when you chain several adapters together:

```rust
let log_lines = vec![
    "INFO: server started",
    "ERROR: connection refused",
    "DEBUG: query executed",
    "ERROR: timeout exceeded",
    "INFO: request handled",
];

let error_report: Vec<String> = log_lines.iter()
    .filter(|line| line.starts_with("ERROR"))
    .enumerate()
    .map(|(index, line)| format!("#{}: {}", index + 1, &line[7..]))
    .collect();
// ["#1: connection refused", "#2: timeout exceeded"]
```

Each adapter transforms the stream one step at a time, and the compiler fuses them into a single pass over the data.

### Lazy evaluation in action

Because adapters are lazy, unused pipelines do nothing:

```rust
let pipeline = (0..1_000_000)
    .map(|x| {
        println!("processing {x}"); // Never prints!
        x * 2
    });
// No output — the iterator has not been consumed

let first: Vec<i32> = pipeline.take(2).collect();
// Now only "processing 0" and "processing 1" print
```

Only two elements are processed, even though the range has a million entries.

## Common Pitfalls
1. **Forgetting to consume the iterator** — Calling `.map().filter()` without a terminal operation does nothing. The compiler warns about unused `Iterator` values.
2. **Using `filter` with double references** — `iter().filter()` passes `&&T` to the predicate. Use `|&&x|` or `|x| **x` to dereference.
3. **Collecting into the wrong type** — `collect()` needs a type annotation. Use turbofish (`::<Vec<_>>`) or annotate the binding.

## Best Practices
1. **Prefer `filter_map` over `filter` + `map`** — When filtering and transforming, `filter_map` is more concise and avoids an intermediate `Option` unwrap.
2. **Use `take` with infinite iterators** — Ranges like `0..` are safe when bounded by `take(n)`.
3. **Profile before optimizing** — Iterator chains usually optimize well, but if you suspect overhead, check the generated assembly with `cargo asm`.

## Summary
- Iterator adapters (map, filter, flat_map, take, skip) are lazy transformations.
- Consumers (collect, sum, for_each, count) drive the chain to completion.
- Chains compile to efficient, single-pass code.
- Always consume your iterator — unused chains do nothing.
- Use type annotations with collect() to specify the output collection.

## Code Examples

**Practical iterator chains — log parsing with filter_map, pairing with zip, and concatenation with chain**

```rust
// A practical log-processing pipeline
fn extract_error_codes(logs: &[&str]) -> Vec<u32> {
    logs.iter()
        .filter(|line| line.starts_with("ERROR"))
        .filter_map(|line| {
            // Extract numeric code from "ERROR[1234]: ..."
            let start = line.find('[')? + 1;
            let end = line.find(']')?;
            line[start..end].parse::<u32>().ok()
        })
        .collect()
}

// zip: pair elements from two iterators
let students = ["Alice", "Bob", "Charlie"];
let grades = [92, 85, 97];

let report: Vec<String> = students.iter()
    .zip(grades.iter())
    .map(|(name, grade)| format!("{name}: {grade}%"))
    .collect();
// ["Alice: 92%", "Bob: 85%", "Charlie: 97%"]

// chain: concatenate two iterators
let combined: Vec<i32> = (1..=3).chain(8..=10).collect();
// [1, 2, 3, 8, 9, 10]
```


## Resources

- [Processing a Series of Items with Iterators](https://doc.rust-lang.org/book/ch13-02-iterators.html) — Official Rust Book chapter on iterators, adapters, and consumers

---

> 📘 *This lesson is part of the [Functional Rust](https://stanza.dev/courses/rust-functional) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*