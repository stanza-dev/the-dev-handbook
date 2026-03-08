---
source_course: "rails-background-jobs"
source_lesson: "rails-background-jobs-retry-strategies"
---

# Retry Strategies

## Introduction
Jobs fail — networks drop, APIs rate-limit, databases lock. Active Job provides `retry_on` and `discard_on` to handle failures gracefully with configurable backoff strategies.

## Key Concepts
- **retry_on**: Rescues a specific exception and re-enqueues after a wait period.
- **discard_on**: Rescues an exception and silently discards the job.
- **Exponential Backoff**: Retry wait times increase with each attempt.

## Real World Context
A payment API job might fail from a temporary network issue — retrying in 30 seconds will likely succeed. But a job failing due to a deleted user should not retry.

## Deep Dive

```ruby
class ApiSyncJob < ApplicationJob
  retry_on StandardError, wait: :polynomially_longer, attempts: 3
  retry_on Timeout::Error, wait: 5.seconds, attempts: 5
  retry_on RateLimitError, wait: 1.minute, attempts: 10
  discard_on ActiveRecord::RecordNotFound
  discard_on ArgumentError

  def perform(record_id)
    record = Record.find(record_id)
    ExternalApi.sync(record)
  end
end
```

### Custom Backoff with Jitter

```ruby
class ResilientJob < ApplicationJob
  retry_on ApiError, wait: ->(executions) {
    (executions ** 2) + rand(30)
  }, attempts: 5
end
```

## Common Pitfalls
1. **Retrying non-transient errors** — Retrying invalid input wastes resources. Use `discard_on`.
2. **Not adding jitter** — Simultaneous retries create storms. Add randomness.

## Best Practices
1. **Be specific with error classes** — Don't rescue `StandardError` when you mean `Timeout::Error`.
2. **Set appropriate attempt limits** — 5 attempts covers most transient failures.

## Summary
- Use `retry_on` for transient errors, `discard_on` for permanent ones.
- `wait: :polynomially_longer` provides increasing backoff.
- Add jitter to prevent retry storms.
- Be specific about which exceptions to handle.

## Code Examples

**Error-specific retry strategies — transient errors retry, permanent errors are discarded**

```ruby
class PaymentSyncJob < ApplicationJob
  retry_on PaymentGateway::TimeoutError, wait: :polynomially_longer, attempts: 5
  retry_on PaymentGateway::RateLimitError, wait: 1.minute, attempts: 10
  discard_on PaymentGateway::InvalidCardError

  def perform(payment_id)
    payment = Payment.find(payment_id)
    PaymentGateway.sync(payment)
  end
end
```


## Resources

- [Retrying or Discarding Failed Jobs](https://guides.rubyonrails.org/active_job_basics.html#retrying-or-discarding-failed-jobs) — Official guide on retry_on and discard_on

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*