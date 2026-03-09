---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-job-error-handling"
---

# Job Error Handling and Retries

## Introduction
Background jobs can fail due to network timeouts, rate limits, or missing records. Active Job provides built-in retry and discard mechanisms to handle failures gracefully without losing work.

## Key Concepts
- **retry_on**: Automatically retries a job when a specific exception occurs, with configurable wait time and attempt limit.
- **discard_on**: Silently discards a job when a specific exception occurs — useful for unrecoverable errors.
- **Idempotency**: Designing jobs so running them multiple times produces the same result as running once.

## Real World Context
A payment processing job that retries on network timeout but discards on invalid card errors ensures payments eventually succeed without charging a declined card repeatedly.

## Deep Dive

### Automatic Retries

```ruby
class ImportDataJob < ApplicationJob
  retry_on StandardError, wait: :polynomially_longer, attempts: 3
  retry_on Timeout::Error, wait: 5.seconds, attempts: 5
  discard_on ActiveJob::DeserializationError
  discard_on ActiveRecord::RecordNotFound

  def perform(import_id)
    import = Import.find(import_id)
    import.process!
  end
end
```

### Custom Retry Logic

```ruby
class ApiCallJob < ApplicationJob
  retry_on ApiRateLimitError, wait: ->(executions) {
    (executions ** 2).seconds  # 1s, 4s, 16s, 64s...
  }, attempts: 5

  def perform(endpoint)
    ApiClient.call(endpoint)
  end
end
```

### Error Callbacks

```ruby
class CriticalJob < ApplicationJob
  rescue_from StandardError do |exception|
    ErrorTracker.report(exception)
    raise exception  # Re-raise to trigger retry
  end

  after_discard do |job, exception|
    AdminMailer.job_failed(job, exception).deliver_later
  end

  def perform(data)
    # ...
  end
end
```

### Idempotency Pattern

```ruby
class ChargeOrderJob < ApplicationJob
  def perform(order_id)
    order = Order.find(order_id)
    return if order.charged?

    order.with_lock do
      return if order.charged?
      charge = PaymentGateway.charge(order.total)
      order.update!(charged: true, charge_id: charge.id)
    end
  end
end
```

## Common Pitfalls
1. **Not designing for idempotency** — Jobs may run multiple times due to retries or queue failures. Always check if the work was already done before proceeding.
2. **Retrying on all errors** — Some errors (invalid arguments, missing records) will never succeed. Use `discard_on` for unrecoverable errors.

## Best Practices
1. **Use polynomially_longer wait** — Exponential backoff avoids overwhelming a failing service with rapid retries.
2. **Lock records before mutating** — Use `with_lock` to prevent race conditions when multiple job executions overlap.

## Summary
- Use `retry_on` with exponential backoff for transient failures.
- Use `discard_on` for errors that will never succeed on retry.
- Design every job to be idempotent — safe to run multiple times.
- Use `with_lock` to prevent race conditions in critical operations.

## Code Examples

**Idempotent payment job — retries on timeout, discards if record is missing, and uses locking to prevent double charges**

```ruby
class PaymentJob < ApplicationJob
  retry_on Timeout::Error, wait: :polynomially_longer, attempts: 3
  discard_on ActiveRecord::RecordNotFound

  def perform(order_id)
    order = Order.find(order_id)
    return if order.charged?  # Idempotency guard

    order.with_lock do
      return if order.charged?
      charge = PaymentGateway.charge(order.total)
      order.update!(charged: true, charge_id: charge.id)
    end
  end
end
```


## Resources

- [Active Job — Exceptions](https://guides.rubyonrails.org/active_job_basics.html#exceptions) — Official Rails guide on retry_on, discard_on, and error handling in Active Job

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*