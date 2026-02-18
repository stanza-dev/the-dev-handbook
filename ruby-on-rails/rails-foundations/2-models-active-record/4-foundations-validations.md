---
source_course: "rails-foundations"
source_lesson: "rails-foundations-validations"
---

# Model Validations

Validations ensure that only valid data gets saved to your database. They run automatically before `save`, `create`, and `update` operations.

## Why Validations?

- **Data integrity**: Ensure required fields are present
- **Business rules**: Enforce application logic
- **User feedback**: Provide helpful error messages
- **Security**: Prevent malicious or malformed data

## Basic Validations

```ruby
class Article < ApplicationRecord
  validates :title, presence: true
  validates :body, presence: true, length: { minimum: 10 }
end
```

Now if you try to save an invalid article:

```ruby
article = Article.new(title: "")
article.save        # => false
article.valid?      # => false
article.errors.full_messages
# => ["Title can't be blank", "Body can't be blank"]
```

## Common Validation Helpers

### presence

Ensures the attribute is not empty:

```ruby
validates :title, presence: true
validates :name, :email, presence: true  # Multiple fields
```

### length

Validates the length of a string:

```ruby
validates :title, length: { minimum: 5 }
validates :bio, length: { maximum: 500 }
validates :password, length: { in: 8..72 }
validates :code, length: { is: 6 }  # Exactly 6 characters
```

### uniqueness

Ensures the value is unique in the database:

```ruby
validates :email, uniqueness: true

# Case-insensitive
validates :email, uniqueness: { case_sensitive: false }

# Scoped uniqueness
validates :title, uniqueness: { scope: :author_id }
```

### format

Matches against a regular expression:

```ruby
validates :email, format: { with: URI::MailTo::EMAIL_REGEXP }

validates :phone, format: {
  with: /\A\d{3}-\d{3}-\d{4}\z/,
  message: "must be in format XXX-XXX-XXXX"
}
```

### numericality

Validates numeric values:

```ruby
validates :price, numericality: true
validates :price, numericality: { greater_than: 0 }
validates :quantity, numericality: { only_integer: true }
validates :age, numericality: { greater_than_or_equal_to: 18 }
```

### inclusion / exclusion

Validates that value is in (or not in) a set:

```ruby
validates :status, inclusion: { in: %w[draft published archived] }
validates :subdomain, exclusion: { in: %w[admin www api] }
```

## Conditional Validations

Run validations only under certain conditions:

```ruby
class User < ApplicationRecord
  validates :password, presence: true, if: :password_required?
  validates :age, presence: true, unless: :minor?

  # Using a lambda
  validates :card_number, presence: true, if: -> { payment_type == 'card' }

  private

  def password_required?
    new_record? || password.present?
  end
end
```

## Custom Validations

Create your own validation methods:

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

## Working with Errors

```ruby
article = Article.new
article.valid?                  # => false

article.errors                  # ActiveModel::Errors object
article.errors[:title]          # => ["can't be blank"]
article.errors.full_messages    # => ["Title can't be blank"]
article.errors.any?             # => true
article.errors.count            # => 1
```

## Resources

- [Active Record Validations](https://guides.rubyonrails.org/active_record_validations.html) — Complete guide to model validations

---

> 📘 *This lesson is part of the [Rails Foundations](https://stanza.dev/courses/rails-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*