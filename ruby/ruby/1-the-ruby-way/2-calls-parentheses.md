---
source_course: "ruby"
source_lesson: "ruby-method-calls-parentheses"
---

# Method Calls & Parentheses

## Introduction

Ruby's method call syntax is uniquely flexible. You can omit parentheses around arguments, which leads to a style called \"Poetry Mode.\" Combined with naming conventions like `?` and `!` suffixes, Ruby methods read almost like natural language. Mastering these conventions is essential for writing and reading idiomatic Ruby.

## Key Concepts

- **Poetry Mode**: The practice of omitting parentheses in method calls when the meaning is unambiguous, making code read like prose.
- **Predicate methods**: Methods ending in `?` that conventionally return a boolean value.
- **Bang methods**: Methods ending in `!` that typically modify the receiver in place or raise exceptions instead of returning nil.

## Real World Context

In a Rails application, Poetry Mode is everywhere. DSL-style calls like `has_many :posts`, `validates :name, presence: true`, and `before_action :authenticate` all rely on omitted parentheses for readability. Knowing when to use and omit parentheses is what makes your Ruby code feel natural to other developers.

## Deep Dive

Ruby allows you to omit parentheses for method calls. This is idiomatic for DSLs and simple getters, but optional for complex logic.

Here is the basic difference between the two styles:

```ruby
puts \"Hello\"      # Idiomatic (Poetry Mode)
puts(\"Hello\")     # Valid but noisier
```

Both lines are equivalent, but the first is preferred in most Ruby code because it reads more naturally.

### When to Use Parentheses

Use parentheses when there are multiple arguments, when there is ambiguity in parsing, or when chaining method calls:

```ruby
# Ambiguous without parens
puts calculate 1, 2  # Error!
puts calculate(1, 2) # Clear

# Chaining
result.map { |x| x * 2 }.sum
```

The first line fails because Ruby cannot determine whether `2` is a second argument to `puts` or to `calculate`. Parentheses resolve the ambiguity.

### Predicate Methods

Methods ending in `?` return a boolean. This is a naming convention, not enforced syntax:

```ruby
[].empty?       # => true
5.positive?     # => true
nil.nil?        # => true
\"hello\".start_with?(\"h\") # => true
```

These predicate methods make conditionals read like questions: `if list.empty?` is immediately clear in intent.

### Bang Methods

Methods ending in `!` (bang methods) usually modify the object in place or raise an exception instead of returning nil:

```ruby
name = \"hello\"
name.upcase!    # modifies name in place
puts name       # => \"HELLO\"

# Compare:
array = []
array.first     # => nil
array.first!    # raises (if it existed)
```

The bang version mutates the original object, while the non-bang version returns a new object. Always be intentional about which one you use.

## Common Pitfalls

1. **Omitting parens with nested method calls** — Writing `puts calculate 1, 2` creates parsing ambiguity. When passing the result of one method as an argument to another, always use parentheses on the inner call.
2. **Assuming bang methods always exist** — Not every method has a `!` counterpart. For example, `Array#first!` does not exist. The bang convention is a guideline, not a guarantee.

## Best Practices

1. **Omit parentheses for zero-argument and DSL-style calls** — Write `user.name` and `has_many :posts` without parentheses to follow Ruby conventions.
2. **Always use parentheses when there is any ambiguity** — If a method takes multiple arguments or its result is passed to another method, add parentheses for clarity.

## Summary

- Ruby lets you omit parentheses for cleaner, more readable code (Poetry Mode).
- Predicate methods (`?`) and bang methods (`!`) follow naming conventions that communicate intent.
- Use parentheses when there is any ambiguity, especially with multiple arguments or chained calls.

## Code Examples

**Demonstrates Ruby's Poetry Mode, predicate methods, and bang methods**

```ruby
# Poetry Mode - omit parentheses for readability
puts "Hello"          # Idiomatic
puts("Hello")         # Valid but noisier

# Predicate methods end in ?
[].empty?             # => true
5.positive?           # => true

# Bang methods modify in place
name = "hello"
name.upcase!          # => "HELLO"
puts name             # => "HELLO"
```


## Resources

- [Ruby Calling Methods Documentation](https://docs.ruby-lang.org/en/4.0/syntax/calling_methods_rdoc.html) — Official Ruby 4.0 reference for method call syntax

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*