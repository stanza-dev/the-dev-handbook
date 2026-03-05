---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-custom-validators"
---

# Building Custom Validators

## Introduction
While Rails provides many built-in validators (presence, uniqueness, format, numericality), you will often need custom validation logic specific to your domain. Rails offers three approaches: custom validation methods, EachValidator classes, and Validator classes with `validates_with`.

## Key Concepts
- **`validate` method**: Registers a custom validation method that can add errors to the record.
- **`EachValidator`**: A reusable validator class that validates a single attribute. Used with the `validates :attr, your_validator: true` syntax.
- **`validates_with`**: Registers a Validator class that can validate the entire record (not just one attribute).
- **`errors.add`**: The method used inside validators to attach error messages to specific attributes.

## Real World Context
Every non-trivial Rails application needs custom validations. Validating that an end date is after a start date, checking email domains against a blocklist, ensuring URLs are well-formed, or verifying business rules like "an order cannot exceed inventory" all require custom validators. Reusable EachValidator classes are especially valuable in large codebases where the same validation appears across multiple models.

## Deep Dive

### Custom Validation Methods

The simplest approach -- define a method and register it with `validate`:

```ruby
class Event < ApplicationRecord
  validate :end_date_after_start_date
  validate :not_in_past, on: :create

  private

  def end_date_after_start_date
    return if start_date.blank? || end_date.blank?
    if end_date <= start_date
      errors.add(:end_date, "must be after start date")
    end
  end

  def not_in_past
    return if start_date.blank?
    if start_date < Date.current
      errors.add(:start_date, "cannot be in the past")
    end
  end
end
```

Note the early return when values are blank. This prevents `NoMethodError` on nil and lets the `presence` validator handle blank checking.

### Custom EachValidator Classes

For reusable validators, create a class that inherits from `ActiveModel::EachValidator`:

```ruby
# app/validators/email_validator.rb
class EmailValidator < ActiveModel::EachValidator
  def validate_each(record, attribute, value)
    return if value.blank?
    unless value =~ URI::MailTo::EMAIL_REGEXP
      record.errors.add(attribute, options[:message] || "is not a valid email")
    end
  end
end

# Usage in model
class User < ApplicationRecord
  validates :email, email: true
  validates :backup_email, email: { message: "must be a valid address" }
end
```

The class name determines the validator key: `EmailValidator` becomes `email:` in the `validates` call. The `options` hash passes any configuration from the model declaration.

### Validator Classes with validates_with

For validations that span multiple attributes:

```ruby
# app/validators/profanity_validator.rb
class ProfanityValidator < ActiveModel::Validator
  BLOCKED_WORDS = %w[bad words here].freeze

  def validate(record)
    options[:fields].each do |field|
      value = record.send(field).to_s.downcase
      if BLOCKED_WORDS.any? { |word| value.include?(word) }
        record.errors.add(field, "contains inappropriate language")
      end
    end
  end
end

# Usage
class Comment < ApplicationRecord
  validates_with ProfanityValidator, fields: [:body, :title]
end
```

## Common Pitfalls
1. **Not handling nil/blank values** -- Always add an early return for blank values. Let the `presence` validator handle that concern separately.
2. **Putting validation logic in callbacks** -- Use `validate` methods, not `before_save` checks. Callbacks that add errors and `throw(:abort)` bypass the standard validation flow.
3. **Hardcoding error messages** -- Use the `options[:message]` pattern to allow models to customize the message.

## Best Practices
1. **Extract reusable validators to `app/validators/`** -- Any validation used in 2+ models should be an EachValidator class.
2. **Follow the naming convention** -- `FooValidator` is used as `validates :attr, foo: true`. Rails expects this naming.
3. **Test validators independently** -- EachValidator classes can be unit-tested without instantiating the full model.

## Summary
- Use `validate :method_name` for one-off custom validations specific to a single model.
- Use `EachValidator` subclasses for reusable per-attribute validators.
- Use `validates_with` for validators that check multiple attributes or the whole record.
- Always handle nil/blank values with an early return.
- Store reusable validators in `app/validators/` and follow the naming convention.

## Code Examples

**A reusable EachValidator for URLs -- validates that the value is a well-formed HTTP(S) URL with a host**

```ruby
# app/validators/url_validator.rb
class UrlValidator < ActiveModel::EachValidator
  def validate_each(record, attribute, value)
    return if value.blank?
    uri = URI.parse(value)
    valid = (uri.is_a?(URI::HTTP) || uri.is_a?(URI::HTTPS)) && uri.host.present?
    record.errors.add(attribute, "is not a valid URL") unless valid
  rescue URI::InvalidURIError
    record.errors.add(attribute, "is not a valid URL")
  end
end

# Usage: validates :website, url: true
```


## Resources

- [Active Record Validations - Custom Validators](https://guides.rubyonrails.org/active_record_validations.html#custom-validators) — Official guide to custom validators

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*