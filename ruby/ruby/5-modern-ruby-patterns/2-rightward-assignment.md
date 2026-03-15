---
source_course: "ruby"
source_lesson: "ruby-rightward-assignment"
---

# Rightward Assignment & One-Line Methods

## Introduction
Modern Ruby introduces syntactic sugar that makes code more expressive and concise. Rightward assignment lets you write data-flow-style code where the result flows into a variable. Endless methods remove boilerplate for simple one-expression methods. And numbered block parameters reduce noise in short blocks. This lesson covers these three features that make modern Ruby feel clean and fluid.

## Key Concepts
- **Rightward Assignment (`=>`)**: Assigns the result of the left-hand expression to the variable on the right. Also used for pattern-matching destructuring.
- **Endless Method**: A method defined with `=` instead of a body and `end`, for single-expression methods (Ruby 3+).
- **Numbered Block Parameters (`_1`, `_2`, ...)**: Implicit block parameters that reference arguments by position, replacing the need for `|x|` in simple blocks.

## Real World Context
In a data pipeline, rightward assignment reads naturally: `fetch_data(url) => raw` then `parse(raw) => result`. Endless methods shine in utility classes with many small methods — a `Converter` class with `def to_cents(dollars) = (dollars * 100).to_i` is immediately readable. These features reduce syntactic noise in code that is already simple, letting the intent shine through.

## Deep Dive

### Rightward Assignment

Ruby allows assigning the result of an expression to a variable on the right side. This creates a natural left-to-right data flow:

```ruby
1 + 2 => result
puts result  # => 3

# Useful with pattern matching
{ name: "Alice", age: 30 } => { name:, age: }
puts name  # => "Alice"
puts age   # => 30
```

The second example combines rightward assignment with hash destructuring. The `{ name:, age: }` pattern extracts those keys into local variables. This is particularly powerful when chaining operations.

### Endless Method Definition

Simple methods can be defined without `end` using the `=` syntax (Ruby 3+):

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

Endless methods are best for truly simple, single-expression methods. If you find yourself using `begin/end` inside one, switch to the traditional syntax instead.

### Numbered Block Parameters

You can access block parameters by number instead of naming them:

```ruby
# Traditional
[1, 2, 3].map { |x| x * 2 }

# With numbered parameters
[1, 2, 3].map { _1 * 2 }          # => [2, 4, 6]

# Multiple parameters
[[1, 2], [3, 4]].map { _1 + _2 }  # => [3, 7]

# Best for short blocks only!
```

Numbered parameters (`_1`, `_2`, etc.) are ideal for very short blocks where naming the parameter adds no clarity. For anything beyond a simple transformation, named parameters are more readable.

## Common Pitfalls
1. **Overusing endless methods for complex logic** — If the method body needs conditionals, multiple statements, or is longer than ~60 characters, the traditional `def...end` form is clearer. Endless methods are for trivially simple expressions.
2. **Using numbered parameters in nested blocks** — Numbered parameters (`_1`) cannot be used in blocks that are nested inside other blocks that also use them. Ruby raises a syntax error because the scope is ambiguous.

## Best Practices
1. **Use rightward assignment for pipeline-style code** — When you have a chain of transformations, rightward assignment reads naturally: `input |> process => output`. Avoid it for simple `x = 1` assignments where the traditional form is clearer.
2. **Reserve numbered parameters for trivially short blocks** — `map { _1 * 2 }` is fine; `select { _1[:age] > _2[:threshold] && _1[:active] }` is not. If you hesitate about readability, use named parameters.

## Summary
- Rightward assignment (`=>`) enables left-to-right data flow and integrates with pattern matching destructuring.
- Endless methods (`def name(args) = expr`) eliminate boilerplate for single-expression methods.
- Numbered block parameters (`_1`, `_2`) reduce noise in short blocks but should not replace named parameters in complex logic.

## Code Examples

**Combining rightward assignment, endless methods, and numbered block parameters**

```ruby
# Rightward assignment with destructuring
{ name: "Alice", role: "admin" } => { name:, role: }
puts "#{name} is #{role}"  # => "Alice is admin"

# Endless method
def double(x) = x * 2

# Numbered block parameters
[1, 2, 3].map { double(_1) }  # => [2, 4, 6]
```


## Resources

- [Pattern Matching (Rightward Assignment)](https://docs.ruby-lang.org/en/4.0/syntax/pattern_matching_rdoc.html) — Official pattern matching docs covering rightward assignment and destructuring

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*