---
source_course: "ruby"
source_lesson: "ruby-array-new-methods"
---

# New Array Methods in Ruby 4.0

## Introduction

Ruby 4.0 introduces new array methods like `Array#rfind` for efficient reverse searching and an optimized `Array#find` override. Combined with existing power methods like `tally`, `compact`, and lazy enumerables, Ruby arrays become even more expressive. This lesson covers the new additions and the most useful existing methods you should know.

## Key Concepts

- **`Array#rfind`**: A new Ruby 4.0 method that searches an array from the end, more efficiently than `reverse.find` or `reverse_each.find`.
- **`Array#find` override**: Ruby 4.0 adds an optimized `Array#find` that is faster than the inherited `Enumerable#find` for arrays.
- **`tally`**: Counts occurrences of each element, returning a hash of element-to-count pairs.
- **Lazy enumerables**: A mode that processes elements one at a time, enabling work with infinite sequences and avoiding intermediate array allocation.

## Real World Context

When processing log files, you often want the last occurrence of an error pattern. Before Ruby 4.0, you had to reverse the array or use `reverse_each`, both of which are wasteful for large arrays. The new `Array#rfind` method searches backward efficiently without creating intermediate objects. Similarly, `tally` replaces the common pattern of `group_by(&:itself).transform_values(&:count)` that developers write repeatedly for analytics and reporting features.

## Deep Dive

### Array#rfind - Efficient Reverse Search

Ruby 4.0 introduces `Array#rfind`, a dedicated method for finding the last element that matches a condition. It is more efficient and more readable than `reverse_each.find`:

```ruby
numbers = [1, 2, 3, 4, 5, 4, 3, 2, 1]

# Ruby 4.0: rfind searches from the end
numbers.rfind { |n| n.even? }  # => 2 (the last even number)

# rfind also accepts a default proc, like find
numbers.rfind(-> { 0 }) { |n| n > 10 }  # => 0 (no match, uses default)

# Old way (less efficient)
numbers.reverse.find { |n| n.even? }       # Creates new reversed array!
numbers.reverse_each.find { |n| n.even? }  # Better but more verbose
```

The `rfind` method avoids creating a reversed copy of the array, making it both faster and more memory-efficient for large collections. It also accepts an optional default proc argument, just like `find`.

### Optimized Array#find

Ruby 4.0 also overrides `Array#find` with a C-level implementation that is faster than the inherited `Enumerable#find`. You do not need to change any code — existing `find` calls on arrays are automatically faster.

### Other Useful Array Methods

Ruby arrays include several powerful methods for common data processing tasks:

```ruby
arr = [1, 2, 3, 4, 5]

# Compact - remove nils
[1, nil, 2, nil, 3].compact  # => [1, 2, 3]

# Tally - count occurrences
[\"a\", \"b\", \"a\", \"c\", \"b\", \"a\"].tally  # => {\"a\"=>3, \"b\"=>2, \"c\"=>1}

# Intersection and Union
[1, 2, 3] & [2, 3, 4]  # => [2, 3]
[1, 2] | [2, 3]        # => [1, 2, 3]

# Difference
[1, 2, 3] - [2]        # => [1, 3]

# Product - cartesian product
[1, 2].product([\"a\", \"b\"])  # => [[1,\"a\"], [1,\"b\"], [2,\"a\"], [2,\"b\"]]
```

The `tally` method is particularly useful for analytics. `compact` is essential when dealing with data that may contain nil values from database queries or API responses.

### Lazy Enumerables

For very large collections, use lazy evaluation to avoid creating intermediate arrays:

```ruby
# Without lazy - creates multiple intermediate arrays
(1..Float::INFINITY).select { |n| n.even? }.first(5)
# Error! Infinite loop

# With lazy - processes elements one at a time
(1..Float::INFINITY).lazy.select { |n| n.even? }.first(5)
# => [2, 4, 6, 8, 10]
```

The `lazy` method returns a lazy enumerator that only computes values as they are needed, making it possible to work with infinite sequences.

## Common Pitfalls

1. **Using `reverse.find` instead of `rfind`** — `reverse` creates a full copy of the array in memory before searching. For large arrays, this is wasteful. Use `rfind` (Ruby 4.0+) for efficient reverse searching.
2. **Forgetting that `lazy` returns an Enumerator, not an Array** — Chaining lazy operations returns a lazy enumerator. You must call `.to_a`, `.first(n)`, or `.force` to materialize the results.

## Best Practices

1. **Use `tally` instead of `group_by` + `count`** — Replace `array.group_by(&:itself).transform_values(&:count)` with `array.tally`. It is shorter, clearer, and faster.
2. **Apply `lazy` to chains with multiple transformations on large data** — When chaining `select`, `map`, and `reject` on large collections, prefix with `lazy` to process elements one at a time and avoid creating intermediate arrays.

## Summary

- Ruby 4.0's `Array#rfind` provides efficient backward searching without reversing the array.
- `Array#find` is now overridden at the C level on Array for better performance than `Enumerable#find`.
- Methods like `tally`, `compact`, and set operators (`&`, `|`, `-`) handle common data processing patterns concisely.
- Lazy enumerables enable processing of infinite or very large sequences by computing values on demand.

## Code Examples

**Demonstrates Array#rfind, tally, and lazy enumerables for efficient collection processing**

```ruby
# Ruby 4.0: rfind searches from the end
numbers = [1, 2, 3, 4, 5, 4, 3, 2, 1]
numbers.rfind { |n| n.even? }  # => 2 (the last even number)

# Tally counts occurrences
["a", "b", "a", "c", "b", "a"].tally
# => {"a"=>3, "b"=>2, "c"=>1}

# Lazy enumerables for infinite sequences
(1..Float::INFINITY).lazy.select(&:even?).first(5)
# => [2, 4, 6, 8, 10]
```


## Resources

- [Ruby Array Documentation](https://docs.ruby-lang.org/en/4.0/Array.html) — Official Ruby 4.0 reference for Array

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*