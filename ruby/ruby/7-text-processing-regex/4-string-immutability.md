---
source_course: "ruby"
source_lesson: "ruby-string-immutability"
---

# Frozen String Literals

Strings are mutable by default in Ruby. This creates memory churn as each string literal creates a new object.

## The Frozen String Pragma

Use this magic comment to make all string literals frozen:

```ruby
# frozen_string_literal: true

str = "hello"
str << " world"  # => FrozenError!

# To modify, create a new string:
str = str + " world"  # New object
str = "#{str} world"  # Interpolation creates new object
```

## Performance Benefits

```ruby
# Without frozen: each creates a NEW string object
1000.times { "hello" }  # 1000 String objects

# With frozen: same object reused
# frozen_string_literal: true
1000.times { "hello" }  # 1 String object
```

## When to Use Frozen Strings

- **Do use** for new projects and libraries
- **Do use** for performance-critical code
- **Don't use** if you need to mutate strings often

## String Deduplication

Ruby 2.5+ automatically deduplicates frozen strings at the VM level:

```ruby
a = "hello".freeze
b = "hello".freeze
a.object_id == b.object_id  # => true
```

## Explicit Freezing

```ruby
# Freeze at assignment
CONSTANT = "important".freeze

# Or use the unary minus operator
CONSTANT = -"important"  # Same as .freeze
```

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*