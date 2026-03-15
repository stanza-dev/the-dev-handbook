---
source_course: "ruby"
source_lesson: "ruby-pattern-matching"
---

# Pattern Matching

## Introduction
Pattern matching, introduced in Ruby 2.7 and progressively stabilized from Ruby 3.0 through 3.2, brings powerful destructuring and conditional logic to the language. It lets you match complex data structures — arrays, hashes, nested objects — in a single expressive statement. This lesson covers the `case/in` syntax, array and hash patterns, guard clauses, and how pattern matching reduces boilerplate in data-heavy code.

## Key Concepts
- **`case/in`**: The pattern matching syntax, distinct from `case/when`. Each `in` branch matches a structural pattern.
- **Array Pattern**: Matches array elements positionally, with support for rest (`*`) and wildcard (`_`) operators.
- **Hash Pattern**: Matches hash keys and optionally binds their values to local variables.
- **Guard Clause**: An `if` condition appended to a pattern for additional filtering.
- **Pin Operator (`^`)**: Matches against the value of an existing variable rather than binding a new one.

## Real World Context
Pattern matching shines when processing API responses, webhook payloads, or any structured data. Instead of writing nested `if` statements to check `response[:status]` and dig into `response[:data]`, you write a single `case/in` that destructures and validates the shape in one step. For example, a payment webhook handler can match `{ event: "charge.succeeded", data: { amount: Integer => amt } }` and immediately have the amount bound to a variable.

## Deep Dive
Pattern matching provides powerful destructuring that replaces verbose conditional logic.

### Basic Case/In Syntax

The `case/in` syntax matches structural patterns against data:

```ruby
case json
in { status: "ok", data: [first, *rest] }
  puts "First item: #{first}"
in { status: "error", message: msg }
  puts "Error: #{msg}"
else
  puts "Unknown format"
end
```

Each `in` branch is tried in order. The first matching pattern wins. Variables like `first`, `rest`, and `msg` are bound automatically.

### Array Patterns

Array patterns match elements by position and support rest and wildcard operators:

```ruby
case [1, 2, 3]
in [a, b, c]
  puts a, b, c        # 1, 2, 3
in [first, *rest]
  puts first, rest    # 1, [2, 3]
in [_, second, _]
  puts second         # 2 (underscores ignore values)
end
```

The `*rest` captures remaining elements into an array. The `_` wildcard matches any single value without binding it to a variable.

### Hash Patterns

Hash patterns match on keys and can bind values or enforce types:

```ruby
user = { name: "Alice", age: 30, city: "NYC" }

case user
in { name: String => n, age: 18.. }
  puts "Adult: #{n}"
in { name: String => n, age: ...18 }
  puts "Minor: #{n}"
end
```

The `String => n` syntax checks that the value is a `String` and binds it to `n`. The `18..` is a range pattern matching any value 18 or greater. Hash patterns are lenient by default — extra keys in the hash are ignored.

### Guard Clauses

You can add an `if` condition to any pattern for more precise matching:

```ruby
case number
in n if n.positive?
  "positive"
in n if n.negative?
  "negative"
in 0
  "zero"
end
```

The guard is evaluated after the pattern matches. If the guard fails, Ruby moves to the next `in` branch.

## Common Pitfalls
1. **Confusing `case/in` with `case/when`** — `case/when` uses `===` for comparison, while `case/in` uses structural pattern matching. Mixing `when` and `in` in the same `case` statement is a syntax error.
2. **Forgetting that hash patterns are lenient** — `in { name: }` matches any hash that *contains* a `:name` key, even if it has many other keys. If you need exact matching, use `in **nil` to reject extra keys: `in { name:, **nil }`.

## Best Practices
1. **Use pattern matching for structured data** — API responses, config hashes, and parsed JSON are ideal candidates. Avoid pattern matching for simple equality checks where `case/when` is clearer.
2. **Combine with Data classes for clean domain modeling** — `Data.define` objects work naturally with pattern matching, giving you type-safe destructuring of value objects.

## Summary
- `case/in` provides structural pattern matching with automatic variable binding.
- Array patterns use positional matching with `*rest` and `_` wildcards; hash patterns match on keys.
- Guard clauses (`if`) add conditional logic on top of structural matches.

## Code Examples

**Destructuring a nested API response with pattern matching**

```ruby
response = { status: "ok", data: [{ id: 1, name: "Alice" }, { id: 2, name: "Bob" }] }

case response
in { status: "ok", data: [first, *] }
  puts "First: #{first[:name]}"  # => "First: Alice"
in { status: "error", message: msg }
  puts "Error: #{msg}"
end
```


## Resources

- [Pattern Matching Guide](https://docs.ruby-lang.org/en/4.0/syntax/pattern_matching_rdoc.html) — Official Pattern Matching syntax guide

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*