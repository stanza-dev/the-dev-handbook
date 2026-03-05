---
source_course: "rails-foundations"
source_lesson: "rails-foundations-validations"
---

# Model Validations

## Introduction

Validations are your application's first line of defense for data integrity. They ensure that only valid, well-formed data gets saved to your database, providing automatic error messages when something is wrong.

## Key Concepts

- **Validation**: A rule defined in a model that must pass before a record can be saved.
- **`validates`**: The DSL method used to declare validations with built-in helpers.
- **`errors`**: An `ActiveModel::Errors` object attached to every model instance, populated when validation fails.
- **Conditional validation**: Validations that only run when certain conditions are met.

## Real World Context

Without validations, your database could contain empty emails, negative prices, or duplicate usernames. Validations catch bad data at the application level before it reaches the database, providing user-friendly error messages instead of cryptic database errors.

## Deep Dive

### Basic Validations

```ruby
class Article < ApplicationRecord
  validates :title, presence: true
  validates :body, presence: true, length: { minimum: 10 }
end

article = Article.new(title: "")
article.valid?  # => false
article.errors.full_messages
# => ["Title can't be blank", "Body can't be blank"]
```

### Common Validation Helpers

```ruby
# Presence
validates :title, presence: true

# Length
validates :title, length: { minimum: 5 }
validates :bio, length: { maximum: 500 }
validates :password, length: { in: 8..72 }

# Uniqueness
validates :email, uniqueness: { case_sensitive: false }

# Format
validates :email, format: { with: URI::MailTo::EMAIL_REGEXP }

# Numericality
validates :price, numericality: { greater_than: 0 }
validates :quantity, numericality: { only_integer: true }

# Inclusion
validates :status, inclusion: { in: %w[draft published archived] }
```

### Conditional Validations

```ruby
class User < ApplicationRecord
  validates :password, presence: true, if: :password_required?
  validates :card_number, presence: true, if: -> { payment_type == 'card' }

  private
  def password_required?
    new_record? || password.present?
  end
end
```

### Custom Validations

```ruby
class Article < ApplicationRecord
  validate :publish_date_cannot_be_in_past

  private
  def publish_date_cannot_be_in_past
    if publish_date.present? && publish_date < Date.today
      errors.add(:publish_date, "can't be in the past")
    end
  end
end
```

### Working with Errors

```ruby
article = Article.new
article.valid?                  # => false
article.errors[:title]          # => ["can't be blank"]
article.errors.full_messages    # => ["Title can't be blank"]
article.errors.count            # => 1
```

## Common Pitfalls

- **Relying only on client-side validation**: Always validate on the server. Client-side validation can be bypassed.
- **Not checking `valid?` before using errors**: The `errors` collection is only populated after calling `valid?`, `save`, or `create`.
- **Uniqueness race conditions**: `validates :email, uniqueness: true` can have race conditions. Add a unique database index as a safety net.

## Best Practices

- Pair model-level uniqueness validations with database-level unique indexes.
- Use `presence: true` on all required fields.
- Write custom error messages that are helpful to end users.

## Summary

- Validations run before `save`, `create`, and `update`, preventing invalid data from being persisted.
- Built-in helpers: `presence`, `length`, `uniqueness`, `format`, `numericality`, `inclusion`.
- Use `errors.full_messages` to get human-readable validation failure descriptions.
- Custom validations use `validate :method_name` with `errors.add` for custom rules.
- Always back uniqueness validations with database indexes.

## Code Examples

**Model validations ensure only valid data is saved. Common helpers include presence, length, and inclusion, with errors accessible via the errors object.**

```ruby
class Article < ApplicationRecord
  validates :title, presence: true, length: { minimum: 5 }
  validates :body, presence: true
  validates :status, inclusion: { in: %w[draft published archived] }
end

article = Article.new(title: "")
article.valid?               # => false
article.errors.full_messages  # => ["Title can't be blank", ...]
```


## Resources

- [Active Record Validations](https://guides.rubyonrails.org/active_record_validations.html) — Complete guide to model validations

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*