---
source_course: "ruby"
source_lesson: "ruby-understanding-enumerable"
---

# The Enumerable Contract

## Introduction

`Enumerable` is Ruby's most powerful module. By defining a single `each` method and including `Enumerable`, your class gains over 50 collection methods for free: `map`, `select`, `reject`, `reduce`, and many more. Understanding this contract is essential because Enumerable is the backbone of data processing in Ruby.

## Key Concepts

- **Enumerable module**: A mixin that provides 50+ collection methods (`map`, `select`, `reduce`, etc.) to any class that implements `each`.
- **The `each` contract**: The single method you must define to make a class Enumerable. It must yield successive elements to a block.
- **Lazy evaluation**: A mode where Enumerable methods process elements one at a time, avoiding intermediate array creation for large or infinite collections.

## Real World Context

When you build a custom collection class in a Ruby application, such as a `Playlist` of songs or a `Team` of members, including Enumerable gives your users the full collection API they already know from Array. This means they can call `playlist.select { |song| song.duration < 300 }` without you writing any filtering logic. Libraries like ActiveRecord use this same pattern to make query results feel like arrays.

## Deep Dive

`Enumerable` is a module that gives you `map`, `select`, `reject`, `reduce`, and 50+ other methods for free. It is mixed into Array, Hash, Set, Range, and more.

### Making a Class Enumerable

To make a class Enumerable, you only need to include the module and define an `each` method that yields items:

```ruby
class Team
  include Enumerable

  def initialize(members)
    @members = members
  end

  def each(&block)
    @members.each(&block)
  end
end

team = Team.new([\"Alice\", \"Bob\", \"Charlie\"])
team.map(&:upcase)     # => [\"ALICE\", \"BOB\", \"CHARLIE\"]
team.select { |m| m.start_with?(\"A\") } # => [\"Alice\"]
```

With just `each` defined, the `Team` class now supports all Enumerable methods, including `map`, `select`, `sort_by`, `min`, `max`, and `reduce`.

### Key Enumerable Methods

The most commonly used Enumerable methods fall into three categories: transformation, filtering, and aggregation:

```ruby
numbers = [1, 2, 3, 4, 5]

# Transformation
numbers.map { |n| n * 2 }     # => [2, 4, 6, 8, 10]
numbers.flat_map { |n| [n, n] } # => [1, 1, 2, 2, ...]

# Filtering
numbers.select { |n| n.even? } # => [2, 4]
numbers.reject { |n| n.even? } # => [1, 3, 5]
numbers.partition { |n| n.even? } # => [[2, 4], [1, 3, 5]]

# Aggregation
numbers.reduce(0) { |sum, n| sum + n } # => 15
numbers.sum                             # => 15 (shortcut)
```

The `partition` method is especially useful when you need to split a collection into two groups in a single pass.

## Common Pitfalls

1. **Forgetting to define `each`** — Including `Enumerable` without defining `each` will result in `NoMethodError` when any Enumerable method is called. The module relies entirely on `each` for iteration.
2. **Using `map` when you need `each`** — `map` builds and returns a new array, while `each` returns the original collection. If you only need side effects (like printing), use `each` to avoid unnecessary memory allocation.

## Best Practices

1. **Prefer `select`/`reject` over manual loops** — Instead of building a filtered array with `each` and `push`, use `select` or `reject`. They are more readable and less error-prone.
2. **Use `reduce` with an initial value** — Always provide an initial accumulator to `reduce` (e.g., `reduce(0)`) to avoid `nil` errors on empty collections and to make the intent clear.

## Summary

- Include `Enumerable` and define `each` to give any class 50+ collection methods for free.
- The three pillars of Enumerable are transformation (`map`), filtering (`select`, `reject`), and aggregation (`reduce`, `sum`).
- Lazy evaluation lets you process infinite or very large collections without memory overhead.

## Code Examples

**Shows how to make a custom class Enumerable by defining each and including the module**

```ruby
class Team
  include Enumerable

  def initialize(members)
    @members = members
  end

  def each(&block)
    @members.each(&block)
  end
end

team = Team.new(["Alice", "Bob", "Charlie"])
team.map(&:upcase)      # => ["ALICE", "BOB", "CHARLIE"]
team.select { |m| m.start_with?("A") }  # => ["Alice"]
```


## Resources

- [Enumerable Module](https://docs.ruby-lang.org/en/4.0/Enumerable.html) — Official Enumerable documentation

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*