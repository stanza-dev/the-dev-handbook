---
source_course: "ruby"
source_lesson: "ruby-custom-exceptions"
---

# Custom Exceptions

## Introduction
Ruby's built-in exception classes cover many common error scenarios, but real applications need domain-specific errors. Custom exception classes let you communicate exactly what went wrong and attach structured context that callers can act on.

## Key Concepts
- **Custom Exception Class**: A class that inherits from `StandardError` (or one of its subclasses) to represent a domain-specific error.
- **Exception Attributes**: Extra data (like field names, status codes, or IDs) attached to an exception via `attr_reader` to provide context beyond just a message string.
- **Exception Hierarchy**: Organizing your custom exceptions into a tree so callers can rescue broadly or narrowly as needed.

## Real World Context
In a web application, you might have a `PaymentDeclined` error, a `UserNotFound` error, and a `RateLimited` error. Each needs different handling: one triggers a customer notification, another returns a 404, and the third implements a retry with backoff. Without custom exceptions, you end up parsing error message strings, which is fragile and error-prone.

## Deep Dive

### Designing Custom Exceptions

Always inherit from `StandardError`, not `Exception`. This ensures your errors are caught by bare `rescue` blocks and do not interfere with system signals.

Here is a typical application error hierarchy:

```ruby
class ApplicationError < StandardError; end
class NotFoundError < ApplicationError; end
class ValidationError < ApplicationError
  attr_reader :field

  def initialize(field, message)
    @field = field
    super("\#{field}: \#{message}")
  end
end
```

`ApplicationError` serves as the base for all your app-specific errors. `NotFoundError` and `ValidationError` branch from it, letting callers rescue at whatever granularity they need.

### Using Custom Exceptions

Custom exceptions integrate naturally with Ruby's `begin/rescue` blocks. Here is how you raise and catch them:

```ruby
def find_user(id)
  user = User.find_by(id: id)
  raise NotFoundError, "User \#{id} not found" unless user
  user
end

def validate_age(age)
  raise ValidationError.new(:age, "must be positive") if age < 0
end

# Catching them
begin
  user = find_user(999)
rescue NotFoundError => e
  puts e.message
rescue ApplicationError => e
  # Catches any ApplicationError subclass
  log_error(e)
end
```

Notice that rescuing `ApplicationError` acts as a catch-all for your domain errors, while rescuing `NotFoundError` lets you handle that specific case differently.

### Adding Context

For errors that originate from external services, you often need structured metadata. Here is an API error class with a status code and response body:

```ruby
class APIError < StandardError
  attr_reader :status_code, :response_body

  def initialize(message, status_code:, response_body: nil)
    @status_code = status_code
    @response_body = response_body
    super(message)
  end
end

raise APIError.new("Rate limited", status_code: 429)
```

The `status_code` and `response_body` attributes give callers the information they need to decide whether to retry, alert, or fail gracefully.

## Common Pitfalls
1. **Inheriting from Exception instead of StandardError** — Your custom error will not be caught by bare `rescue` blocks and may interfere with signal handling. Always inherit from `StandardError` or one of its descendants.
2. **Flat exception hierarchies** — Creating dozens of unrelated exception classes makes it impossible to rescue a group of related errors. Use a base class like `ApplicationError` to enable broad rescues.

## Best Practices
1. **Create a base ApplicationError** — Having a single root for your domain errors lets you rescue all application errors in one clause when you need a safety net.
2. **Attach structured context via attr_reader** — Instead of encoding information in the message string, use attributes so callers can programmatically inspect the error.

## Summary
- Custom exceptions inherit from `StandardError` and represent domain-specific error conditions.
- Organize them into a hierarchy so callers can rescue broadly or narrowly.
- Attach structured context (field names, status codes, response bodies) as attributes rather than encoding them in message strings.

## Code Examples

**Demonstrates a custom ValidationError with structured attributes that callers can inspect programmatically.**

```ruby
class ApplicationError < StandardError; end

class ValidationError < ApplicationError
  attr_reader :field, :code

  def initialize(field, message, code: nil)
    @field = field
    @code = code
    super("#{field}: #{message}")
  end
end

begin
  raise ValidationError.new(:email, "is invalid", code: :format)
rescue ValidationError => e
  puts e.message  # => email: is invalid
  puts e.field    # => email
  puts e.code     # => format
end
```


## Resources

- [Exception Class](https://docs.ruby-lang.org/en/4.0/Exception.html) — Official Ruby documentation for the Exception class and its subclasses.

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*