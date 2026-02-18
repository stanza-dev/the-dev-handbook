---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-job-error-handling"
---

# Job Error Handling

Background jobs can fail. Proper error handling ensures reliability.

## Automatic Retries

```ruby
class ImportDataJob < ApplicationJob
  # Retry 3 times with exponential backoff
  retry_on StandardError, wait: :polynomially_longer, attempts: 3
  
  # Specific error handling
  retry_on Timeout::Error, wait: 5.seconds, attempts: 5
  retry_on ActiveRecord::RecordNotFound, wait: 1.minute, attempts: 3
  
  # Don't retry certain errors
  discard_on ActiveJob::DeserializationError
  
  def perform(import_id)
    import = Import.find(import_id)
    import.process!
  end
end
```

## Custom Retry Logic

```ruby
class ApiCallJob < ApplicationJob
  retry_on ApiRateLimitError, wait: ->(executions) {
    # Exponential backoff: 1s, 4s, 16s, 64s...
    (executions ** 2).seconds
  }, attempts: 5
  
  def perform(endpoint)
    ApiClient.call(endpoint)
  end
end
```

## Error Callbacks

```ruby
class CriticalJob < ApplicationJob
  # Called on each retry
  rescue_from StandardError do |exception|
    ErrorTracker.report(exception)
    raise exception  # Re-raise to trigger retry
  end
  
  # Called after all retries exhausted
  after_discard do |job, exception|
    AdminMailer.job_failed(job, exception).deliver_later
  end
  
  def perform(data)
    # ...
  end
end
```

## Idempotency

Jobs may run multiple times. Design for idempotency:

```ruby
class ChargeOrderJob < ApplicationJob
  def perform(order_id)
    order = Order.find(order_id)
    
    # Skip if already charged
    return if order.charged?
    
    # Use transaction with locking
    order.with_lock do
      return if order.charged?
      
      charge = PaymentGateway.charge(order.total)
      order.update!(
        charged: true,
        charge_id: charge.id
      )
    end
  end
end
```

## Timeouts

```ruby
class LongRunningJob < ApplicationJob
  # Set timeout (backend-specific)
  sidekiq_options timeout: 5.minutes
  
  def perform(id)
    Timeout.timeout(4.minutes) do
      # Long operation
    end
  end
end
```

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*