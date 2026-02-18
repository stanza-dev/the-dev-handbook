---
source_course: "rust-functional"
source_lesson: "rust-func-iterator-adapters"
---

# Iterator Adapters

Adapters transform iterators lazily (nothing happens until consumed).

## Common Adapters

```rust
let numbers = vec![1, 2, 3, 4, 5];

// map: Transform each element
let doubled: Vec<_> = numbers.iter().map(|x| x * 2).collect();
// [2, 4, 6, 8, 10]

// filter: Keep matching elements
let evens: Vec<_> = numbers.iter().filter(|x| *x % 2 == 0).collect();
// [2, 4]

// filter_map: Filter and transform in one
let parsed: Vec<i32> = ["1", "two", "3"].iter()
    .filter_map(|s| s.parse().ok())
    .collect();
// [1, 3]

// flat_map: Map then flatten
let flattened: Vec<_> = [[1, 2], [3, 4]].iter()
    .flat_map(|arr| arr.iter())
    .collect();
// [1, 2, 3, 4]
```

## Positioning Adapters

```rust
let nums = 1..=10;

// take: First N elements
let first_3: Vec<_> = nums.clone().take(3).collect();  // [1, 2, 3]

// skip: Skip first N elements
let skip_3: Vec<_> = nums.clone().skip(3).collect();  // [4, 5, ..., 10]

// take_while / skip_while
let while_small: Vec<_> = nums.clone().take_while(|&x| x < 5).collect();  // [1, 2, 3, 4]
```

## Combining Adapters

```rust
let data = vec!["hello", "world", "rust", "programming"];

let result: Vec<_> = data.iter()
    .filter(|s| s.len() > 4)      // Keep long words
    .map(|s| s.to_uppercase())    // Uppercase
    .take(2)                       // First 2
    .collect();
// ["HELLO", "WORLD"]
```

## Lazy Evaluation

```rust
let iter = (0..1_000_000)
    .map(|x| {
        println!("Processing {x}");  // Never prints!
        x * 2
    });
// Nothing happens yet - iterators are lazy!

let first = iter.take(1).collect::<Vec<_>>();
// Now only "Processing 0" prints
```

See [Iterators](https://doc.rust-lang.org/book/ch13-02-iterators.html).

## Code Examples

**Iterator adapter examples**

```rust
// Practical iterator chain
fn process_logs(logs: &[&str]) -> Vec<String> {
    logs.iter()
        .filter(|line| !line.starts_with('#'))  // Skip comments
        .filter(|line| !line.is_empty())        // Skip empty
        .map(|line| line.trim())                // Trim whitespace
        .filter(|line| line.contains("ERROR")) // Only errors
        .enumerate()                            // Add index
        .map(|(i, line)| format!("{}: {}", i + 1, line))
        .collect()
}

// zip: Pair up two iterators
let names = ["Alice", "Bob", "Charlie"];
let scores = [95, 87, 92];

let results: Vec<_> = names.iter()
    .zip(scores.iter())
    .map(|(name, score)| format!("{name}: {score}"))
    .collect();
// ["Alice: 95", "Bob: 87", "Charlie: 92"]

// chain: Concatenate iterators
let combined: Vec<_> = (1..=3).chain(7..=9).collect();
// [1, 2, 3, 7, 8, 9]

// cycle: Repeat infinitely
let pattern: Vec<_> = [1, 2, 3].iter().cycle().take(8).collect();
// [1, 2, 3, 1, 2, 3, 1, 2]
```


## Resources

- [Iterators](https://doc.rust-lang.org/book/ch13-02-iterators.html) — Rust Book chapter on iterators

---

> 📘 *This lesson is part of the [Functional Rust](https://stanza.dev/courses/rust-functional) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*