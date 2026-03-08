---
source_course: "rails-background-jobs"
source_lesson: "rails-background-jobs-error-callbacks"
---

# Error Callbacks and Notifications

## Introduction
When jobs fail permanently, you need to know. Active Job provides `rescue_from` and `after_discard` callbacks for custom error handling.

## Key Concepts
- **rescue_from**: Catches exceptions for custom handling. Re-raise to allow retries.
- **after_discard**: Runs after a job is permanently discarded.

## Real World Context
A payment job that exhausts retries needs immediate attention. An import that fails on one record should log and continue.

## Deep Dive

### rescue_from

```ruby
class CriticalJob < ApplicationJob
  retry_on StandardError, attempts: 3

  rescue_from StandardError do |exception|
    Rails.logger.error "Job failed: #{exception.message}"
    ErrorTracker.notify(exception)
    raise exception  # Re-raise to trigger retry_on
  end

  def perform(data)
    # Job logic
  end
end
```

### after_discard

```ruby
class PaymentJob < ApplicationJob
  retry_on PaymentGateway::Error, attempts: 5

  after_discard do |job, exception|
    AdminMailer.job_permanently_failed(
      job_class: job.class.name,
      arguments: job.arguments,
      error: exception&.message
    ).deliver_later(queue: :critical)
  end

  def perform(payment_id)
    Payment.find(payment_id).then { |p| PaymentGateway.charge(p) }
  end
end
```

## Common Pitfalls
1. **Forgetting to re-raise in rescue_from** — Without re-raising, the job is considered successful.
2. **Sending failure notifications on the same backed-up queue** — Use a separate `:critical` queue.

## Best Practices
1. **Integrate error tracking** — Sentry, Honeybadger, etc.
2. **Update affected records on failure** — Set a `failed` status so dashboards show what happened.

## Summary
- `rescue_from` handles exceptions — re-raise to allow retries.
- `after_discard` runs when all retries are exhausted.
- Integrate error tracking for automated alerting.
- Update records to reflect failure status.

## Code Examples

**after_discard callback updates the record and alerts admins**

```ruby
class ImportJob < ApplicationJob
  retry_on StandardError, attempts: 3

  after_discard do |job, exception|
    import = Import.find(job.arguments.first)
    import.update!(status: :failed, error: exception&.message)
    AdminNotifier.alert("Import #{import.id} permanently failed")
  end

  def perform(import_id)
    Import.find(import_id).then { |i| ImportService.process(i) }
  end
end
```


## Resources

- [Active Job — Exceptions](https://guides.rubyonrails.org/active_job_basics.html#exceptions) — Official guide on exception handling

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*