---
source_course: "ruby"
source_lesson: "ruby-truthiness-falsiness"
---

# Truthiness and Falsiness

## Introduction

Ruby takes a beautifully simple approach to boolean logic: only `false` and `nil` are falsy, and absolutely everything else is truthy. This rule trips up developers coming from JavaScript or Python, but once internalized, it makes conditional logic cleaner and more predictable.

## Key Concepts

- **Falsy values**: In Ruby, only `false` and `nil` evaluate as false in boolean contexts. There are no other falsy values.
- **Safe navigation operator (`&.`)**: An operator that calls a method on a receiver only if it is not nil, returning nil instead of raising `NoMethodError`.
- **Conditional assignment (`||=`)**: An idiom that assigns a value only if the variable is currently nil or false.

## Real World Context

In a web application, you frequently deal with optional values from user input, database queries, and API responses. Ruby's truthiness rules let you write concise guard clauses like `return unless user` and default-value patterns like `name = params[:name] || \"Guest\"`. The safe navigation operator `&.` eliminates entire chains of nil checks when traversing nested objects.

## Deep Dive

In Ruby, **only two values are falsy**: `false` and `nil`. Everything else is truthy, including values that are falsy in other languages:

- `0` (zero)
- `\"\"` (empty string)
- `[]` (empty array)
- `{}` (empty hash)

This example demonstrates that both `0` and an empty string are truthy:

```ruby
if 0
  puts \"0 is truthy!\" # This runs!
end

if \"\"
  puts \"Empty string is truthy!\" # This runs too!
end
```

Both messages print because neither `0` nor `\"\"` is `false` or `nil`.

### The Safe Navigation Operator

Use `&.` to call methods on potentially nil objects without raising an error:

```ruby
user = nil
user&.name  # => nil (no error)
user.name   # => NoMethodError!

# Chaining
user&.address&.city # => nil if any is nil
```

The `&.` operator short-circuits the entire chain when it encounters nil, making deep traversals safe and concise.

### Boolean Coercion

Use `!!` (double negation) to convert any value to its boolean equivalent:

```ruby
!!nil   # => false
!!false # => false
!!0     # => true
!!\"\"    # => true
```

This is useful when you need an explicit `true` or `false` value, such as when returning from a predicate method.

### Practical Patterns

Ruby's truthiness rules enable several powerful idioms:

```ruby
# Default values with ||
name = params[:name] || \"Anonymous\"

# Conditional assignment
@user ||= User.find(id) # Only assigns if @user is nil/false

# Guard clauses
return unless user # Returns if user is nil or false
```

The `||=` operator is especially common for memoization: it computes and caches a value on first access, then reuses the cached result on subsequent calls.

## Common Pitfalls

1. **Using `||` for defaults when `false` is a valid value** — The pattern `value = opts[:flag] || true` will override an explicit `false` because `false` is falsy. Use `fetch` or `nil?` checks instead: `value = opts.fetch(:flag, true)`.
2. **Expecting `0` or `\"\"` to be falsy** — Developers from JavaScript or Python often write `if count` expecting it to fail when count is `0`. In Ruby, use `if count.zero?` or `if count > 0` for explicit numeric checks.

## Best Practices

1. **Use the safe navigation operator for nil-prone chains** — Write `user&.profile&.avatar_url` instead of nested `if` statements or `try` calls. It is cleaner and communicates intent.
2. **Prefer guard clauses over nested conditionals** — `return unless user` at the top of a method is more readable than wrapping the entire body in `if user`.

## Summary

- Only `false` and `nil` are falsy in Ruby; everything else (including `0`, `\"\"`, and `[]`) is truthy.
- The safe navigation operator `&.` prevents `NoMethodError` on nil receivers.
- Idioms like `||=` and guard clauses leverage truthiness for concise, readable code.

## Code Examples

**Demonstrates Ruby's truthiness rules, the safe navigation operator, and common default-value idioms**

```ruby
# Only false and nil are falsy
!!nil     # => false
!!false   # => false
!!0       # => true   (unlike JavaScript!)
!!""      # => true   (unlike Python!)

# Safe navigation operator
user = nil
user&.name          # => nil (no NoMethodError)

# Default values and conditional assignment
name = params[:name] || "Anonymous"
@user ||= User.find(id)
```


## Resources

- [Ruby NilClass Documentation](https://docs.ruby-lang.org/en/4.0/NilClass.html) — Official Ruby 4.0 reference for NilClass

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*