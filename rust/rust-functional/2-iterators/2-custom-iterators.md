---
source_course: "rust-functional"
source_lesson: "rust-func-custom-iterators"
---

# Implementing Custom Iterators

## Introduction
Any struct can become iterable by implementing the `Iterator` trait's single required method: `next()`. Once implemented, you get over 70 adapter and consumer methods for free. This lesson shows how to build your own iterators and integrate them with Rust's iterator ecosystem.

## Key Concepts
- **`Iterator` trait**: Requires only `type Item` and `fn next(&mut self) -> Option<Self::Item>`. Return `Some(value)` for each element and `None` when exhausted.
- **`IntoIterator` trait**: Allows your type to work with `for` loops. Implement it to convert your collection into an iterator.
- **Zero-cost iteration**: Custom iterators compile to the same efficient code as hand-written loops.

## Real World Context
Libraries like `serde`, `regex`, and `tokio` define custom iterators everywhere. A `Matches` iterator yields regex matches, a `Lines` iterator yields text lines, and a database cursor yields rows. Implementing `Iterator` on your domain types lets users chain your data with the full power of Rust's iterator ecosystem.

## Deep Dive

### The Iterator trait

The trait is surprisingly simple:

```rust
trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
    // 70+ provided methods: map, filter, fold, sum, etc.
}
```

You implement `next()`, and Rust provides everything else. Let's build a counter:

```rust
struct Counter {
    current: u32,
    max: u32,
}

impl Counter {
    fn up_to(max: u32) -> Self {
        Counter { current: 0, max }
    }
}

impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<u32> {
        if self.current < self.max {
            self.current += 1;
            Some(self.current)
        } else {
            None // Iteration complete
        }
    }
}
```

Once `next()` returns `None`, the iterator is exhausted. Now you can use all adapter methods:

```rust
let sum: u32 = Counter::up_to(5).sum(); // 1+2+3+4+5 = 15
let evens: Vec<u32> = Counter::up_to(10)
    .filter(|x| x % 2 == 0)
    .collect(); // [2, 4, 6, 8, 10]
```

### Iterators over references

When iterating over borrowed data, the `Item` type includes a lifetime:

```rust
struct SlidingWindow<'a, T> {
    data: &'a [T],
    size: usize,
    position: usize,
}

impl<'a, T> Iterator for SlidingWindow<'a, T> {
    type Item = &'a [T];

    fn next(&mut self) -> Option<Self::Item> {
        if self.position + self.size <= self.data.len() {
            let window = &self.data[self.position..self.position + self.size];
            self.position += 1;
            Some(window)
        } else {
            None
        }
    }
}
```

The lifetime `'a` ensures the iterator cannot outlive the data it references.

### IntoIterator for for-loop support

To let your type work with `for item in my_collection`, implement `IntoIterator`:

```rust
struct TaskList {
    tasks: Vec<String>,
}

impl IntoIterator for TaskList {
    type Item = String;
    type IntoIter = std::vec::IntoIter<String>;

    fn into_iter(self) -> Self::IntoIter {
        self.tasks.into_iter()
    }
}

// Now you can write:
let list = TaskList { tasks: vec!["build".into(), "test".into()] };
for task in list {
    println!("Running: {task}");
}
```

## Common Pitfalls
1. **Forgetting to return None** — An iterator that never returns `None` creates an infinite loop when consumed by `collect()` or `sum()`. Always have a termination condition.
2. **Mutable state bugs** — Since `next()` takes `&mut self`, ensure your state transitions are correct. Off-by-one errors in position tracking are common.
3. **Implementing Iterator instead of IntoIterator** — If your type is a collection, implement `IntoIterator` to convert it. Implement `Iterator` on a separate cursor/state struct.

## Best Practices
1. **Keep `next()` simple** — Complex logic in `next()` makes iterators hard to debug. Factor business logic into helper methods.
2. **Provide size hints** — Override `size_hint()` when you know the iterator length. This lets `collect()` pre-allocate the right capacity.
3. **Implement `IntoIterator` for `&MyType` and `&mut MyType`** — This gives users `for item in &collection` (shared) and `for item in &mut collection` (mutable) patterns.

## Summary
- Implement `next()` on the `Iterator` trait to create a custom iterator.
- Return `Some(value)` for each element and `None` when done.
- Over 70 methods (map, filter, fold, sum, etc.) come for free.
- Use `IntoIterator` to enable `for` loop support on collections.
- Custom iterators have zero overhead compared to hand-written loops.

## Code Examples

**An infinite Fibonacci iterator demonstrating how custom iterators compose with standard adapters like take, filter, and sum**

```rust
// A Fibonacci iterator — infinite, but safe with take()
struct Fibonacci {
    current: u64,
    next: u64,
}

impl Fibonacci {
    fn new() -> Self {
        Fibonacci { current: 0, next: 1 }
    }
}

impl Iterator for Fibonacci {
    type Item = u64;

    fn next(&mut self) -> Option<u64> {
        let value = self.current;
        self.current = self.next;
        self.next = value.saturating_add(self.current);
        Some(value) // Infinite iterator — always returns Some
    }
}

// Use with adapters — take() makes it finite
let first_ten: Vec<u64> = Fibonacci::new().take(10).collect();
assert_eq!(first_ten, vec![0, 1, 1, 2, 3, 5, 8, 13, 21, 34]);

// Sum of first 20 Fibonacci numbers
let total: u64 = Fibonacci::new().take(20).sum();
println!("Sum of first 20: {total}"); // Output: Sum of first 20: 17710

// Even Fibonacci numbers under 1000
let even_fibs: Vec<u64> = Fibonacci::new()
    .take_while(|&n| n < 1000)
    .filter(|n| n % 2 == 0)
    .collect();
// [0, 2, 8, 34, 144, 610]
```


## Resources

- [std::iter::Iterator trait](https://doc.rust-lang.org/std/iter/trait.Iterator.html) — Complete API reference for the Iterator trait with all 70+ provided methods

---

> 📘 *This lesson is part of the [Functional Rust](https://stanza.dev/courses/rust-functional) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*