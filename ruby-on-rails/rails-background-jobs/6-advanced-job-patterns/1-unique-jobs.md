---
source_course: "rails-background-jobs"
source_lesson: "rails-background-jobs-unique-jobs"
---

# Unique Jobs and Deduplication

## Introduction
Without uniqueness controls, the same job can run multiple times concurrently or be enqueued repeatedly. Solid Queue provides built-in concurrency controls, and you can implement custom deduplication for any backend.

## Key Concepts
- **Unique Job**: A job that prevents duplicate concurrent execution.
- **limits_concurrency**: Solid Queue's built-in uniqueness control.
- **Lock Pattern**: Using Redis or database locks to prevent duplicate execution.

## Real World Context
A user clicks 'Send Email' five times rapidly. Without uniqueness, they receive five emails.

## Deep Dive

### Solid Queue Concurrency Controls

```ruby
class ProcessOrderJob < ApplicationJob
  limits_concurrency to: 1, key: ->(order_id) { order_id }

  def perform(order_id)
    # Only one instance runs per order_id at a time
  end
end
```

### DIY Unique Jobs with Redis

```ruby
class UniqueJob < ApplicationJob
  before_perform :acquire_lock
  after_perform :release_lock
  discard_on DuplicateJobError

  def unique_key
    [self.class.name, arguments].join(':')
  end

  private

  def acquire_lock
    acquired = Redis.current.set("job_lock:#{unique_key}", job_id, nx: true, ex: 1.hour.to_i)
    raise DuplicateJobError unless acquired
  end

  def release_lock
    Redis.current.del("job_lock:#{unique_key}") if Redis.current.get("job_lock:#{unique_key}") == job_id
  end
end
```

## Common Pitfalls
1. **Not releasing locks on failure** — Use `after_perform` and `ensure` blocks.
2. **Lock timeout too short** — If the job takes longer than the lock TTL, another instance starts.

## Best Practices
1. **Use Solid Queue's limits_concurrency when possible** — Built-in, no Redis needed.
2. **Set lock TTL longer than the job's expected duration.**

## Summary
- Solid Queue provides `limits_concurrency` for built-in uniqueness.
- For other backends, use Redis-based locking patterns.
- Always set appropriate lock timeouts.
- Use `after_perform` to release locks.

## Code Examples

**Solid Queue concurrency control — only one job per order_id at a time**

```ruby
class ProcessOrderJob < ApplicationJob
  limits_concurrency to: 1, key: ->(order_id) { order_id }

  def perform(order_id)
    order = Order.find(order_id)
    OrderProcessor.process(order)
  end
end
```


## Resources

- [Solid Queue Concurrency Controls](https://github.com/rails/solid_queue#concurrency-controls) — Concurrency controls documentation

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*