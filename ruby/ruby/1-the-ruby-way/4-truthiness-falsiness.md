---
source_course: "ruby"
source_lesson: "ruby-truthiness-falsiness"
---

# Ruby's Simple Falsy Rules

In Ruby, **only two values are falsy**: `false` and `nil`. Everything else is truthy, including:

- `0` (zero)
- `""` (empty string)
- `[]` (empty array)
- `{}` (empty hash)

```ruby
if 0
  puts "0 is truthy!" # This runs!
end

if ""
  puts "Empty string is truthy!" # This runs too!
end
```

## The Safe Navigation Operator

Use `&.` to call methods on potentially nil objects:

```ruby
user = nil
user&.name  # => nil (no error)
user.name   # => NoMethodError!

# Chaining
user&.address&.city # => nil if any is nil
```

## Boolean Coercion

Use `!!` to convert any value to its boolean equivalent:

```ruby
!!nil   # => false
!!false # => false
!!0     # => true
!!""    # => true
```

## Practical Patterns

```ruby
# Default values with ||
name = params[:name] || "Anonymous"

# Conditional assignment
@user ||= User.find(id) # Only assigns if @user is nil/false

# Guard clauses
return unless user # Returns if user is nil or false
```

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*