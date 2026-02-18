---
source_course: "ruby"
source_lesson: "ruby-exception-best-practices"
---

# Production Error Handling

## 1. Be Specific About What You Catch

```ruby
# Bad - too broad
begin
  process_order
rescue => e
  log(e)
end

# Good - catch specific errors
begin
  process_order
rescue PaymentDeclined => e
  notify_customer(e)
rescue InventoryError => e
  backorder_items(e)
rescue => e
  # Log unexpected errors
  Sentry.capture_exception(e)
  raise
end
```

## 2. Don't Swallow Exceptions Silently

```ruby
# Bad
begin
  dangerous_operation
rescue
  # Nothing - silent failure!
end

# Good
begin
  dangerous_operation
rescue => e
  logger.error(e.full_message)
  # Decide: re-raise, return default, or handle
end
```

## 3. Use Exception Chaining (cause)

```ruby
class WrappedError < StandardError; end

begin
  external_api_call
rescue APIError => e
  raise WrappedError, "External service failed", cause: e
end

# Access the original:
error.cause  # => the original APIError
```

## 4. Use throw/catch for Flow Control

When you need non-local return without being an "error":

```ruby
result = catch(:found) do
  data.each do |row|
    row.each do |cell|
      throw(:found, cell) if cell.matches?(criteria)
    end
  end
  nil  # Default if not found
end
```

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*