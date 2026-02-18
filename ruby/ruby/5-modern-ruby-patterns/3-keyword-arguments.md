---
source_course: "ruby"
source_lesson: "ruby-keyword-arguments"
---

# Keyword Arguments

Modern Ruby enforces strict separation between positional and keyword arguments.

## Basic Keyword Arguments

```ruby
def call_api(url:, method: :get, timeout: 30)
  puts "#{method.upcase} #{url} (timeout: #{timeout})"
end

call_api(url: "/users")                # Uses defaults
call_api(url: "/users", method: :post)  # Override default
```

## Capturing Extra Keywords

Use `**` to capture additional keyword arguments:

```ruby
def request(url:, **options)
  puts "URL: #{url}"
  puts "Options: #{options}"
end

request(url: "/api", timeout: 30, retries: 3)
# URL: /api
# Options: {:timeout=>30, :retries=>3}
```

## Forwarding Arguments

Ruby 3+ provides `...` for complete argument forwarding:

```ruby
def wrapper(...)
  puts "Before"
  inner_method(...)
  puts "After"
end

# Also forward blocks
def with_logging(&)
  puts "Start"
  yield
  puts "End"
end
```

## Ruby 4.0: No More *nil to Array Conversion

In Ruby 4.0, `*nil` no longer calls `nil.to_a`:

```ruby
# Ruby 3.x: *nil => [] (called nil.to_a)
# Ruby 4.0: *nil => nil (no conversion)

# Be explicit when needed:
array = value.nil? ? [] : [*value]
```

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*