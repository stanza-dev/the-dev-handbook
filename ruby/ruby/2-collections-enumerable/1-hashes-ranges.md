---
source_course: "ruby"
source_lesson: "ruby-arrays-hashes-ranges"
---

# Arrays, Hashes & Ranges

## Introduction

Arrays, hashes, and ranges are the three foundational collection types in Ruby. Arrays store ordered lists, hashes map keys to values, and ranges represent intervals. Ruby provides expressive literal syntax for all three, making collection creation concise and readable.

## Key Concepts

- **Array**: An ordered, integer-indexed collection that can hold objects of any type. Created with `[]` or special `%w` and `%i` literals.
- **Hash**: A key-value collection where keys can be any object. The modern syntax uses `key: value` for symbol keys.
- **Range**: An interval between two values, either inclusive (`..`) or exclusive (`...`) of the end value.

## Real World Context

In any Ruby application, you work with these collections constantly. A REST API controller might receive params as a hash, query the database to get an array of records, and paginate results using a range. Knowing the literal syntax shortcuts like `%w[]` for word arrays saves time and reduces visual noise in configuration files and test data.

## Deep Dive

Ruby provides powerful literals for collections.

### Arrays

Arrays support several creation syntaxes:

```ruby
# Standard literal
numbers = [1, 2, 3]

# Word array literal
words = %w[apple banana cherry]  # [\"apple\", \"banana\", \"cherry\"]

# Symbol array literal
symbols = %i[a b c]              # [:a, :b, :c]
```

The `%w` literal is especially useful for lists of strings without special characters, eliminating the need for quotes and commas.

### Hashes

Hashes support multiple key types and syntaxes:

```ruby
# Symbol keys (preferred modern style)
user = { name: \"Alice\", age: 30 }
user[:name] # => \"Alice\"

# String keys
data = { \"key\" => \"value\" }
data[\"key\"] # => \"value\"

# Mixed keys
mixed = { :symbol => 1, \"string\" => 2, 42 => \"number key\" }
```

The `name: value` syntax is syntactic sugar for `:name => value` and is the preferred style for symbol keys in modern Ruby.

### Ranges

Ranges define intervals and support both inclusive and exclusive end values:

```ruby
(1..5).to_a   # [1, 2, 3, 4, 5] (inclusive)
(1...5).to_a  # [1, 2, 3, 4]    (exclusive of end)

# Endless and beginless ranges (Ruby 2.6+/2.7+)
(1..)         # 1 to infinity
(..10)        # negative infinity to 10

# Range membership
(1..10).include?(5)  # => true
(1..10).cover?(5)    # => true (faster for large ranges)
```

Note that `include?` iterates through the range to check membership, while `cover?` uses comparison operators and is much faster for numeric or string ranges.

## Common Pitfalls

1. **Confusing `..` and `...`** — Two dots (`..`) includes the end value, three dots (`...`) excludes it. `(1..5)` includes 5 but `(1...5)` does not. This is a frequent source of off-by-one errors.
2. **Using `Hash.new([])` instead of a block** — `Hash.new([])` shares one array across all keys. Use `Hash.new { |h, k| h[k] = [] }` to create a fresh array for each missing key.

## Best Practices

1. **Use `%w` and `%i` for static lists** — Write `%w[admin editor viewer]` instead of `[\"admin\", \"editor\", \"viewer\"]` for cleaner, more scannable code.
2. **Prefer `cover?` over `include?` for ranges** — When checking membership in a numeric or date range, `cover?` is O(1) while `include?` may iterate the entire range.

## Summary

- Ruby provides concise literal syntax for arrays (`[]`, `%w`, `%i`), hashes (`{ key: value }`), and ranges (`..`, `...`).
- Hashes support any object as a key, and the modern symbol-key syntax is preferred.
- Ranges support inclusive, exclusive, endless, and beginless forms for flexible interval definitions.

## Code Examples

**Demonstrates Ruby's collection literals for arrays, hashes, and ranges**

```ruby
# Array literals
numbers = [1, 2, 3]
words = %w[apple banana cherry]  # => ["apple", "banana", "cherry"]
symbols = %i[a b c]              # => [:a, :b, :c]

# Hash with symbol keys
user = { name: "Alice", age: 30 }
user[:name]  # => "Alice"

# Ranges
(1..5).to_a   # => [1, 2, 3, 4, 5] (inclusive)
(1...5).to_a  # => [1, 2, 3, 4]    (exclusive)
```


## Resources

- [Array Docs](https://docs.ruby-lang.org/en/4.0/Array.html) — Official Array documentation

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*