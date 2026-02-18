---
source_course: "rails-active-record-mastery"
source_lesson: "rails-active-record-mastery-custom-validators"
---

# Building Custom Validators

While Rails provides many built-in validators, you'll often need custom validation logic specific to your domain.

## Custom Validation Methods

The simplest approach - define a method and use `validate`:

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

## Custom Validator Classes

For reusable validators, create a class:

### EachValidator - For Single Attributes

```ruby
# app/validators/email_validator.rb
class EmailValidator < ActiveModel::EachValidator
  def validate_each(record, attribute, value)
    return if value.blank?

    unless value =~ URI::MailTo::EMAIL_REGEXP
      record.errors.add(attribute, options[:message] || "is not a valid email")
    end

    # Check for disposable email domains
    if options[:reject_disposable]
      domain = value.split("@").last
      if DisposableEmailDomain.exists?(domain: domain)
        record.errors.add(attribute, "cannot be a disposable email")
      end
    end
  end
end

# Usage in model
class User < ApplicationRecord
  validates :email, email: true
  validates :backup_email, email: { reject_disposable: true }
end
```

### More Custom EachValidators

```ruby
# app/validators/url_validator.rb
class UrlValidator < ActiveModel::EachValidator
  def validate_each(record, attribute, value)
    return if value.blank?

    begin
      uri = URI.parse(value)
      valid = uri.is_a?(URI::HTTP) || uri.is_a?(URI::HTTPS)
      valid &&= uri.host.present?
    rescue URI::InvalidURIError
      valid = false
    end

    unless valid
      record.errors.add(attribute, options[:message] || "is not a valid URL")
    end
  end
end

# app/validators/future_date_validator.rb
class FutureDateValidator < ActiveModel::EachValidator
  def validate_each(record, attribute, value)
    return if value.blank?

    unless value > Date.current
      record.errors.add(attribute, options[:message] || "must be in the future")
    end
  end
end

# Usage
class Event < ApplicationRecord
  validates :website, url: true
  validates :starts_on, future_date: { message: "cannot be in the past" }
end
```

### Validator Classes with validates_with

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

## Resources

- [Active Record Validations - Custom Validators](https://guides.rubyonrails.org/active_record_validations.html#custom-validators) — Official guide to custom validators

---

> 📘 *This lesson is part of the [Active Record Mastery](https://stanza.dev/courses/rails-active-record-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*