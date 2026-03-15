---
source_course: "ruby"
source_lesson: "ruby-exception-best-practices"
---

# Exception Best Practices

## Introduction
Knowing the syntax of `begin/rescue/ensure` is only half the battle. How you structure error handling across an application determines whether bugs are caught early, failures are visible, and production systems stay debuggable. This lesson covers the patterns that separate robust Ruby code from brittle code.

## Key Concepts
- **Exception Specificity**: Rescuing the narrowest exception class that matches the failure you expect.
- **Exception Swallowing**: Catching an error and doing nothing with it, which hides bugs and makes debugging impossible.
- **Exception Chaining (cause)**: Wrapping a low-level exception in a higher-level one while preserving the original as `cause`.
- **throw/catch**: A non-exception flow control mechanism for non-local returns that do not represent errors.

## Real World Context
In a production e-commerce system, an order processing pipeline might fail due to payment decline, inventory shortage, or an unexpected database error. Each failure needs different handling: customer notification, backorder creation, or alerting the on-call engineer. Broad rescue blocks that swallow all errors mean the team discovers failures hours later from customer complaints instead of monitoring alerts.

## Deep Dive

### 1. Be Specific About What You Catch

Catching broad exceptions masks bugs and makes it impossible to distinguish expected failures from unexpected ones. Here is the problematic pattern:

```ruby
# Bad - too broad
begin
  process_order
rescue => e
  log(e)
end
```

This catches every `StandardError`, including bugs like `NoMethodError` that indicate programming errors. Instead, rescue specific errors and handle each appropriately:

```ruby
# Good - catch specific errors
begin
  process_order
rescue PaymentDeclined => e
  notify_customer(e)
rescue InventoryError => e
  backorder_items(e)
rescue => e
  # Log unexpected errors and re-raise
  Sentry.capture_exception(e)
  raise
end
```

The final `rescue => e` acts as a safety net for truly unexpected errors, but it re-raises so the error propagates and becomes visible in monitoring.

### 2. Don't Swallow Exceptions Silently

A silent rescue is one of the most dangerous patterns in Ruby. It hides failures completely:

```ruby
# Bad - silent failure
begin
  dangerous_operation
rescue
  # Nothing - silent failure!
end
```

At minimum, log the error. Then decide whether to re-raise, return a default, or take corrective action:

```ruby
# Good - at least log the error
begin
  dangerous_operation
rescue => e
  logger.error(e.full_message)
  # Decide: re-raise, return default, or handle
end
```

The `full_message` method includes the backtrace, which is invaluable for debugging production issues.

### 3. Use Exception Chaining (cause)

When you catch a low-level exception and raise a higher-level one, use `cause` to preserve the original error. This creates a chain of exceptions that tells the full story:

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

Exception chaining lets your public API raise clean, domain-specific errors while still preserving the technical details of what went wrong underneath.

### 4. Use throw/catch for Flow Control

When you need a non-local return that is not an error condition, use `throw`/`catch` instead of exceptions. This is semantically different from error handling:

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

Unlike exceptions, `throw`/`catch` does not create a backtrace and carries no error semantics. It is Ruby's way of doing a multi-level break.

## Common Pitfalls
1. **Swallowing exceptions silently** — An empty rescue block hides failures completely. Even if you cannot handle the error, always log it and consider re-raising.
2. **Using exceptions for flow control** — Raising and rescuing exceptions is expensive because Ruby builds a backtrace. For non-error scenarios like searching nested data, use `throw`/`catch` or early returns.

## Best Practices
1. **Re-raise unexpected errors after logging** — A catch-all `rescue => e` at the bottom of your rescue chain should log the error to your monitoring system and then `raise` to let it propagate.
2. **Use exception chaining to preserve context** — When wrapping a low-level error in a domain-specific one, always pass `cause:` so debugging the root cause remains possible.

## Summary
- Rescue the narrowest exception class you expect, and re-raise unexpected errors after logging them.
- Never swallow exceptions silently — at minimum log the error with its full backtrace.
- Use exception chaining (`cause`) to wrap low-level errors in domain-specific ones while preserving the original for debugging.

## Code Examples

**Shows how to wrap a low-level network error in a domain-specific exception while preserving the original cause for debugging.**

```ruby
# Exception chaining: wrapping low-level errors
class ServiceError < StandardError; end

def call_external_service
  # Simulate an API failure
  raise Net::ReadTimeout, "connection timed out"
rescue Net::ReadTimeout => e
  raise ServiceError, "Payment gateway unavailable", cause: e
end

begin
  call_external_service
rescue ServiceError => e
  puts e.message       # => Payment gateway unavailable
  puts e.cause.message # => connection timed out
  puts e.cause.class   # => Net::ReadTimeout
end
```


## Resources

- [Exception Class](https://docs.ruby-lang.org/en/4.0/Exception.html) — Official Ruby documentation for Exception, including the cause method and exception chaining.

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*