---
source_course: "rust-functional"
source_lesson: "rust-func-iterator-new-methods"
---

# New Iterator Methods in Recent Rust

## Introduction
Rust's standard library continually adds new iterator-related methods. This lesson covers notable additions from Rust 1.87 through 1.94, including `Vec::extract_if`, `array_windows`, and `Peekable::next_if_map`. These methods reduce boilerplate and express common patterns more concisely.

## Key Concepts
- **`Vec::extract_if`** (stabilized 1.87): Removes elements matching a predicate in place and yields them as an iterator.
- **`<[T]>::array_windows`** (stabilized 1.94): Yields overlapping fixed-size array references over a slice, similar to `windows()` but returning arrays instead of slices.
- **`Peekable::next_if_map`** (stabilized 1.94): Takes ownership of the next element, applies a function returning `Result<R, I::Item>`, and consumes the element only if the function returns `Ok`. Returning `Err(item)` puts the element back.

## Real World Context
Before `extract_if`, removing elements conditionally from a `Vec` required awkward `retain`/`drain` combinations or manual index management. `array_windows` eliminates the need for `windows().map(|w| <[T; N]>::try_from(w).unwrap())` patterns. These methods make common operations concise and safe.

## Deep Dive

### Vec::extract_if (Rust 1.87)

`extract_if` drains elements matching a predicate while keeping the rest. It is like `retain` but also gives you the removed elements:

```rust
let mut scores = vec![95, 42, 88, 31, 76, 15];

// Remove failing scores (below 50) and collect them
let failing: Vec<i32> = scores.extract_if(.., |&mut score| score < 50).collect();

assert_eq!(failing, vec![42, 31, 15]);
assert_eq!(scores, vec![95, 88, 76]); // Only passing scores remain
```

The first argument is a range specifying which portion of the vec to scan. `..` means the entire vec. The predicate receives `&mut T` so you can inspect each element.

### Slice::array_windows (Rust 1.94)

`array_windows` yields overlapping windows as fixed-size arrays rather than slices:

```rust
let prices = [100.0, 102.5, 98.3, 105.7, 103.2];

// Compute price changes between consecutive days
let changes: Vec<f64> = prices
    .array_windows::<2>()
    .map(|[prev, curr]| curr - prev)
    .collect();

assert_eq!(changes, vec![2.5, -4.2, 7.4, -2.5]);
```

Compared to the older `windows(2)` method, `array_windows` returns `&[T; N]` instead of `&[T]`, enabling pattern matching on the array size at compile time.

### Peekable::next_if_map (Rust 1.94)

`next_if_map` takes ownership of the next element and passes it to a function returning `Result<R, I::Item>`. If the function returns `Ok(r)`, the element is consumed and `Some(r)` is returned. If it returns `Err(item)`, the element is put back and `None` is returned:

```rust
let mut iter = vec!["42", "hello", "99"].into_iter().peekable();

// Consume and parse the next element only if it is a valid number
let parsed: Option<i32> = iter.next_if_map(|s| {
    s.parse::<i32>().map_err(|_| s) // Ok(42) — consumed
});
assert_eq!(parsed, Some(42));

// "hello" is not a number, so it is put back via Err
let not_parsed: Option<i32> = iter.next_if_map(|s| {
    s.parse::<i32>().map_err(|_| s) // Err("hello") — put back
});
assert_eq!(not_parsed, None);

// "hello" is still the next element
assert_eq!(iter.next(), Some("hello"));
```

The `Result`-based return type lets you "un-consume" the element by returning it via `Err`. There is also a companion `next_if_map_mut` method that takes `&mut I::Item` and returns `Option<R>` instead.

## Common Pitfalls
1. **Forgetting to consume `extract_if`** — Like all iterator adapters, `extract_if` is lazy. If you do not consume it (e.g., with `collect()` or `for_each()`), no elements are removed.
2. **Wrong const generic for `array_windows`** — The window size is a const generic parameter. Using `array_windows::<1>()` yields single-element arrays, which is rarely useful.
3. **Confusing `next_if` with `next_if_map`** — `next_if` takes a predicate and returns the element unchanged. `next_if_map` takes a function returning `Result<R, I::Item>` and consumes only on `Ok`, putting the element back on `Err`.

## Best Practices
1. **Use `extract_if` instead of `retain` + separate collection** — It is more efficient (single pass) and more expressive.
2. **Prefer `array_windows` for fixed-size patterns** — When you know the window size at compile time, `array_windows` gives better ergonomics and type safety than `windows()`.
3. **Combine `peekable` methods judiciously** — `next_if`, `next_if_map`, and `peek` are powerful for tokenizers and parsers. Use the one that fits your transformation needs.

## Summary
- `Vec::extract_if` (1.87) removes and yields matching elements in a single pass.
- `array_windows` (1.94) yields overlapping windows as compile-time-sized arrays.
- `Peekable::next_if_map` (1.94) conditionally consumes and transforms the next element via `Result<R, I::Item>`.
- These methods reduce boilerplate for common collection operations.
- Always consume lazy iterators like `extract_if` or they do nothing.

## Code Examples

**Combining extract_if for outlier removal with array_windows for rolling averages — a practical data analysis pipeline**

```rust
// Combining new methods in a data processing pipeline
fn analyze_sensor_readings(mut readings: Vec<f64>) -> (Vec<f64>, Vec<f64>) {
    // Remove outliers (values beyond 3 standard deviations)
    let mean = readings.iter().sum::<f64>() / readings.len() as f64;
    let std_dev = (readings.iter()
        .map(|x| (x - mean).powi(2))
        .sum::<f64>() / readings.len() as f64)
        .sqrt();

    let threshold = 3.0 * std_dev;
    let outliers: Vec<f64> = readings
        .extract_if(.., |&mut r| (r - mean).abs() > threshold)
        .collect();

    // Compute rolling averages with array_windows on the cleaned data
    let rolling_avg: Vec<f64> = readings
        .array_windows::<3>()
        .map(|[a, b, c]| (a + b + c) / 3.0)
        .collect();

    (outliers, rolling_avg)
}
```


## Resources

- [Vec::extract_if documentation](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.extract_if) — API reference for Vec::extract_if with examples and edge case documentation

---

> 📘 *This lesson is part of the [Functional Rust](https://stanza.dev/courses/rust-functional) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*