---
source_course: "rails-background-jobs"
source_lesson: "rails-background-jobs-concurrency-patterns"
---

# Concurrency and Rate Limiting

## Introduction
When jobs interact with external APIs that have rate limits, or with shared resources that cannot handle concurrent access, you need concurrency controls and rate limiting patterns.

## Key Concepts
- **Rate Limiting**: Restricting how many jobs execute per time period.
- **Semaphore Pattern**: Limiting concurrent access to a shared resource.
- **Throttling**: Spacing out job execution to avoid overwhelming a service.

## Real World Context
A third-party API allows 100 requests per minute. If you enqueue 1,000 sync jobs, you need to throttle them to stay under the limit.

## Deep Dive

### Throttling with Delays

```ruby
class ApiSyncCoordinatorJob < ApplicationJob
  def perform(record_ids)
    record_ids.each_with_index do |id, index|
      # Space out API calls: 100 per minute = 1 every 0.6s
      ApiSyncJob.set(wait: (index * 0.6).seconds).perform_later(id)
    end
  end
end
```

### Solid Queue Concurrency Limits

```ruby
class ExternalApiJob < ApplicationJob
  # Max 5 concurrent jobs hitting this API
  limits_concurrency to: 5, key: -> { "external_api" }

  def perform(record_id)
    ExternalApi.sync(Record.find(record_id))
  end
end
```

### Re-enqueue on Rate Limit

```ruby
class ApiCallJob < ApplicationJob
  retry_on RateLimitError, wait: ->(executions) {
    # Use the Retry-After header if available
    60.seconds
  }, attempts: 10

  def perform(id)
    ApiClient.call(id)
  end
end
```

## Common Pitfalls
1. **Not respecting Retry-After headers** — APIs tell you how long to wait. Use that value.
2. **Global concurrency limits that are too restrictive** — Measure actual API limits before setting.

## Best Practices
1. **Use Solid Queue's limits_concurrency for API rate limiting** — Simple and database-backed.
2. **Log rate limit events** — Track how often you hit limits to inform capacity planning.

## Summary
- Throttle jobs with `set(wait:)` for rate-limited APIs.
- Use `limits_concurrency` for concurrency caps.
- Handle rate limit errors with `retry_on` and appropriate wait times.

## Code Examples

**Combining concurrency limits with rate limit retries**

```ruby
class ExternalApiJob < ApplicationJob
  limits_concurrency to: 5, key: -> { "external_api" }
  retry_on RateLimitError, wait: 60.seconds, attempts: 10

  def perform(record_id)
    ExternalApi.sync(Record.find(record_id))
  end
end
```


## Resources

- [Solid Queue Concurrency Controls](https://github.com/rails/solid_queue#concurrency-controls) — Concurrency controls

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*