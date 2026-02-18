---
source_course: "rails-background-processing"
source_lesson: "rails-background-jobs-retry-strategies"
---

# Retry Strategies

Jobs fail. Good retry strategies ensure eventual success.

## Active Job Retries

```ruby
class ApiSyncJob < ApplicationJob
  # Retry 3 times with exponential backoff
  retry_on StandardError, wait: :polynomially_longer, attempts: 3
  
  # Specific error handling
  retry_on Timeout::Error, wait: 5.seconds, attempts: 5
  retry_on RateLimitError, wait: 1.minute, attempts: 10
  
  # Give up on certain errors
  discard_on ActiveRecord::RecordNotFound
  discard_on ArgumentError
  
  def perform(record_id)
    record = Record.find(record_id)
    ExternalApi.sync(record)
  end
end
```

## Custom Wait Strategies

```ruby
class ResilientJob < ApplicationJob
  # Custom backoff calculation
  retry_on ApiError, wait: ->(executions) {
    (executions ** 2) + rand(30)  # Jitter to prevent thundering herd
  }, attempts: 5
  
  # Fixed wait
  retry_on NetworkError, wait: 30.seconds, attempts: 3
  
  # Polynomial backoff (default)
  retry_on StandardError, wait: :polynomially_longer, attempts: 5
  # Wait times: 3s, 18s, 83s, 258s, 625s
end
```

## Sidekiq-Specific Options

```ruby
class SidekiqJob < ApplicationJob
  sidekiq_options(
    retry: 5,           # Max retries
    backtrace: 20,      # Lines of backtrace to store
    dead: true,         # Move to dead queue after retries
    lock: :until_executed  # Sidekiq Enterprise unique jobs
  )
end
```

## Manual Retry Logic

```ruby
class ComplexJob < ApplicationJob
  def perform(id, attempt = 1)
    result = try_operation(id)
    
    if result.failed? && attempt < 3
      self.class.set(wait: attempt.minutes).perform_later(id, attempt + 1)
    elsif result.failed?
      handle_final_failure(id)
    end
  end
end
```

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-processing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*