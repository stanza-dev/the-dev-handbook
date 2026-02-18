---
source_course: "ruby"
source_lesson: "ruby-rightward-assignment"
---

# Rightward Assignment

Ruby allows assigning the result of an expression to a variable on the right:

```ruby
1 + 2 => result
puts result  # => 3

# Useful with pattern matching
{ name: "Alice", age: 30 } => { name:, age: }
puts name  # => "Alice"
puts age   # => 30
```

## Endless Method Definition

Define simple methods without `end` (Ruby 3+):

```ruby
# Traditional
def square(x)
  x * x
end

# Endless (no 'end' needed)
def square(x) = x * x

# Works with more complex expressions
def greet(name) = "Hello, #{name}!"

# With multiple statements? Use begin/end or traditional
def complex(x) = begin
  y = x * 2
  y + 1
end
```

## Numbered Block Parameters

Access block parameters by number:

```ruby
# Traditional
[1, 2, 3].map { |x| x * 2 }

# With numbered parameters
[1, 2, 3].map { _1 * 2 }          # => [2, 4, 6]

# Multiple parameters
[[1, 2], [3, 4]].map { _1 + _2 }  # => [3, 7]

# Best for short blocks only!
```

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*