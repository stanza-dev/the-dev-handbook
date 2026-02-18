---
source_course: "rails-background-processing"
source_lesson: "rails-background-jobs-error-callbacks"
---

# Error Callbacks

Monitor and respond to job failures.

## Active Job Error Handling

```ruby
class CriticalJob < ApplicationJob
  # Called on each retry
  rescue_from StandardError do |exception|
    Rails.logger.error("Job failed: #{exception.message}")
    ErrorTracker.notify(exception, job_id: job_id)
    raise exception  # Re-raise to trigger retry
  end
  
  # Called when all retries exhausted
  after_discard do |job, exception|
    AdminMailer.job_permanently_failed(
      job_class: job.class.name,
      arguments: job.arguments,
      error: exception.message
    ).deliver_later(queue: :critical)
  end
  
  def perform(data)
    # Job logic
  end
end
```

## Sidekiq Death Handlers

```ruby
# config/initializers/sidekiq.rb
Sidekiq.configure_server do |config|
  config.death_handlers << ->(job, exception) do
    # Job exhausted all retries
    Rails.logger.error("Job #{job['class']} died: #{exception.message}")
    
    # Notify team
    Slack.notify(
      channel: '#alerts',
      message: "🚨 Job permanently failed: #{job['class']}"
    )
    
    # Track metric
    StatsD.increment('jobs.dead', tags: ["class:#{job['class']}"])
  end
end
```

## Error-Specific Handling

```ruby
class PaymentJob < ApplicationJob
  rescue_from PaymentDeclined do |exception|
    # Don't retry, notify customer
    PaymentMailer.declined(exception.payment).deliver_later
    # Don't re-raise - job is "successful" (handled)
  end
  
  rescue_from PaymentGatewayError do |exception|
    ErrorTracker.notify(exception)
    raise exception  # Retry - transient error
  end
  
  def perform(order_id)
    order = Order.find(order_id)
    PaymentService.charge(order)
  end
end
```

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-processing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*