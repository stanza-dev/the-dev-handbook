---
source_course: "ruby"
source_lesson: "ruby-set-core-class"
---

# Set: A Core Class in Ruby 4.0

## Introduction

Ruby 4.0 promoted `Set` from the standard library to a core class, meaning you no longer need `require 'set'`. Sets provide unordered collections of unique values with O(1) membership testing and mathematical set operations. If you need uniqueness or fast lookups, Set is your tool.

## Key Concepts

- **Set**: An unordered collection that automatically deduplicates entries and provides O(1) membership testing.
- **Set operations**: Mathematical operations like union (`|`), intersection (`&`), difference (`-`), and symmetric difference (`^`).
- **Core class promotion**: In Ruby 4.0, Set moved from `require 'set'` to being available by default, just like Array and Hash.

## Real World Context

When building a permissions system, you might store a user's roles as a Set and check membership with `roles.include?(:admin)` in O(1) time. When computing which features are available to a user, set intersection between the user's plan features and the requested features gives you the answer in a single operation. Sets also shine in data deduplication tasks like removing duplicate email addresses from an import.

## Deep Dive

In Ruby 4.0, `Set` has been promoted from the standard library to a core class. No more `require 'set'` needed!

Creating a set automatically removes duplicates:

```ruby
# Ruby 4.0 - no require needed!
fruits = Set.new([\"apple\", \"banana\", \"apple\"])
puts fruits # => Set[\"apple\", \"banana\"]
```

The duplicate `\"apple\"` is silently removed during construction.

### Set Characteristics

Sets are unordered collections of unique values with fast O(1) membership testing that automatically deduplicate entries:

```ruby
set = Set.new([1, 2, 2, 3, 3, 3])
set.to_a  # => [1, 2, 3]

# Adding elements
set.add(4)     # => Set adds 4
set << 5       # => Same as add
set.add(1)     # => No effect, 1 already exists
```

Adding a value that already exists is a no-op, making Sets ideal for accumulating unique items.

### Common Set Operations

Sets support the full suite of mathematical set operations:

```ruby
a = Set[1, 2, 3]
b = Set[2, 3, 4]

a | b          # Union: Set[1, 2, 3, 4]
a & b          # Intersection: Set[2, 3]
a - b          # Difference: Set[1]
a ^ b          # Symmetric difference: Set[1, 4]

a.subset?(b)   # => false
a.superset?(Set[1, 2]) # => true
```

These operators make it easy to combine, compare, and filter collections without writing loops.

### When to Use Set vs Array

| Use Set when | Use Array when |
|--------------|----------------|
| You need unique values | Order matters |
| Frequent membership checks | You need duplicates |
| Set operations (union, etc.) | Index-based access |

## Common Pitfalls

1. **Expecting ordered iteration** — Sets do not guarantee insertion order in the way Arrays do. If you need deterministic ordering, convert to an array first with `set.sort` or `set.to_a`.
2. **Using mutable objects as set members** — If you mutate an object after adding it to a Set, it may break the internal hash lookup. Prefer immutable values (strings, symbols, integers) as set members.

## Best Practices

1. **Choose Set over Array when uniqueness matters** — If you are calling `array.uniq` frequently, use a Set from the start. It enforces uniqueness automatically and avoids O(n) deduplication passes.
2. **Use set operations instead of manual loops** — Instead of iterating two arrays to find common elements, use `set_a & set_b`. It is faster and communicates intent clearly.

## Summary

- Set is a core class in Ruby 4.0, requiring no `require` statement.
- Sets provide O(1) membership testing and automatic deduplication.
- Mathematical set operations (`|`, `&`, `-`, `^`) replace manual loop-based collection comparisons.

## Code Examples

**Demonstrates Set creation, deduplication, and mathematical set operations in Ruby 4.0**

```ruby
# Set is a core class in Ruby 4.0 — no require needed
fruits = Set.new(["apple", "banana", "apple"])
fruits  # => Set["apple", "banana"]

# Set operations
a = Set[1, 2, 3]
b = Set[2, 3, 4]
a | b   # Union: Set[1, 2, 3, 4]
a & b   # Intersection: Set[2, 3]
a - b   # Difference: Set[1]
a ^ b   # Symmetric difference: Set[1, 4]
```


## Resources

- [Set Class](https://docs.ruby-lang.org/en/4.0/Set.html) — Official Set documentation

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*