---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-normalizations"
---

# Attribute Normalizations

## Introduction

Rails 7.1 introduced `normalizes`, which automatically transforms attribute values before validation. Instead of `before_validation` callbacks to strip whitespace or downcase emails, you declare normalizations directly on the model.

## Key Concepts

- **`normalizes`**: A class method that registers a transformation to run automatically when an attribute is set.
- **Applied on assignment**: Normalizations run immediately when you assign a value, not just before save.
- **Applied in queries**: Normalizations are also applied to query values for consistent lookups.

## Real World Context

Every application needs normalization. Emails should be lowercase. Phone numbers need consistent formatting. Before `normalizes`, this required `before_validation` callbacks. The `normalizes` method centralizes this with a clean, declarative API.

## Deep Dive

### Basic Normalization

```ruby
class User < ApplicationRecord
  normalizes :email, with: -> e { e.strip.downcase }
end
```

This normalizes the email whenever it is assigned:

```ruby
user = User.new(email: "  ALICE@Example.COM  ")
user.email  # => "alice@example.com" — normalized immediately!
```

### Multiple Normalizations

```ruby
class Article < ApplicationRecord
  normalizes :title, with: -> t { t.strip.squeeze(" ") }
  normalizes :slug, with: -> s { s.strip.downcase.gsub(/\s+/, "-") }
end
```

Each attribute gets its own normalization lambda.

### Normalizations in Queries

Normalizations are applied to query values too:

```ruby
User.find_by(email: "  ALICE@Example.COM  ")
# SQL: SELECT * FROM users WHERE email = 'alice@example.com'
```

This prevents mismatches between stored values and query parameters.

### Comparison with before_validation

```ruby
# Old approach
class User < ApplicationRecord
  before_validation :normalize_email
  private
  def normalize_email
    self.email = email&.strip&.downcase
  end
end

# New approach (Rails 7.1+)
class User < ApplicationRecord
  normalizes :email, with: -> e { e.strip.downcase }
end
```

The `normalizes` approach is shorter, declarative, applies on assignment, and normalizes query values.

## Common Pitfalls

- **Normalizing to nil accidentally**: If your lambda returns nil for blank strings, the attribute becomes nil.
- **Expensive normalizations**: The lambda runs on every assignment. Keep it lightweight.
- **Not using normalizes in queries**: If you normalize on save but query with raw input, you get mismatches. `normalizes` handles this automatically.

## Best Practices

- Use `normalizes` for simple transformations: strip, downcase, formatting.
- Keep normalization lambdas pure (no side effects).
- Replace `before_validation` normalization callbacks with `normalizes` when on Rails 7.1+.

## Summary

- `normalizes` declares automatic attribute transformations that run on assignment.
- Normalizations are also applied to query values for consistent lookups.
- Use `normalizes` instead of `before_validation` callbacks.
- Keep normalization lambdas simple and pure.
- Handle nil carefully — normalizations skip nil by default.

## Code Examples

**normalizes — transforms values on assignment and in queries, replacing before_validation callbacks.**

```ruby
class User < ApplicationRecord
  normalizes :email, with: -> e { e.strip.downcase }
  normalizes :name, with: -> n { n.strip.squeeze(" ") }
end

user = User.new(email: "  ALICE@Example.COM  ")
user.email  # => "alice@example.com" — normalized on assignment

# Queries are normalized too
User.find_by(email: "  Alice@EXAMPLE.com  ")
# SQL: WHERE email = 'alice@example.com'
```


## Resources

- [Active Record Normalizations](https://api.rubyonrails.org/v8.0/classes/ActiveRecord/Normalization/ClassMethods.html) — API documentation for the normalizes method

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*