---
source_course: "ruby"
source_lesson: "ruby-pattern-matching"
---

# Pattern Matching

Introduced in Ruby 2.7 and matured through Ruby 4.0, pattern matching provides powerful destructuring.

## Basic Case/In Syntax

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

## Array Patterns

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

## Hash Patterns

```ruby
user = { name: "Alice", age: 30, city: "NYC" }

case user
in { name: String => n, age: 18.. }
  puts "Adult: #{n}"
in { name: String => n, age: ...18 }
  puts "Minor: #{n}"
end
```

## Guard Clauses

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

## Resources

- [Pattern Matching Guide](https://docs.ruby-lang.org/en/4.0/syntax/pattern_matching_rdoc.html) — Official Pattern Matching syntax guide

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*