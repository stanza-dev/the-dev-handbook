---
source_course: "ruby"
source_lesson: "ruby-custom-exceptions"
---

# Designing Custom Exceptions

Creating custom exception classes makes debugging easier. Always inherit from `StandardError`, not `Exception`.

```ruby
class ApplicationError < StandardError; end
class NotFoundError < ApplicationError; end
class ValidationError < ApplicationError
  attr_reader :field

  def initialize(field, message)
    @field = field
    super("#{field}: #{message}")
  end
end
```

## Using Custom Exceptions

```ruby
def find_user(id)
  user = User.find_by(id: id)
  raise NotFoundError, "User #{id} not found" unless user
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

## Adding Context

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

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*