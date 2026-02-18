---
source_course: "ruby"
source_lesson: "ruby-arrays-hashes-ranges"
---

# Literals

Ruby provides powerful literals for collections.

## Arrays

```ruby
# Standard literal
numbers = [1, 2, 3]

# Word array literal
words = %w[apple banana cherry]  # ["apple", "banana", "cherry"]

# Symbol array literal
symbols = %i[a b c]              # [:a, :b, :c]
```

## Hashes

```ruby
# Symbol keys (preferred modern style)
user = { name: "Alice", age: 30 }
user[:name] # => "Alice"

# String keys
data = { "key" => "value" }
data["key"] # => "value"

# Mixed keys
mixed = { :symbol => 1, "string" => 2, 42 => "number key" }
```

## Ranges

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

## Resources

- [Array Docs](https://docs.ruby-lang.org/en/4.0/Array.html) — Official Array documentation

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*