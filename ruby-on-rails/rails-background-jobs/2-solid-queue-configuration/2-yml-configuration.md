---
source_course: "rails-background-jobs"
source_lesson: "rails-background-jobs-queue-yml-configuration"
---

# Configuring Workers and Dispatchers

## Introduction
Solid Queue uses a YAML configuration file to define how jobs are processed. Understanding dispatchers and workers lets you tune concurrency and prioritize queues.

## Key Concepts
- **Worker**: A process that picks up and executes jobs from queues.
- **Dispatcher**: A process that polls for scheduled jobs and moves them to the ready queue.
- **Concurrency (threads)**: Number of jobs a worker can execute simultaneously.

## Real World Context
A typical setup: one dispatcher, dedicated workers for critical queues, general workers for default queues.

## Deep Dive

### Default Configuration

```yaml
# config/queue.yml
default: &default
  dispatchers:
    - polling_interval: 1
      batch_size: 500
  workers:
    - queues: "*"
      threads: 3
      processes: <%= ENV.fetch("JOB_CONCURRENCY", 1) %>
      polling_interval: 0.1
```

### Queue-Specific Workers

```yaml
production:
  dispatchers:
    - polling_interval: 1
      batch_size: 500
  workers:
    - queues: critical
      threads: 3
      processes: 1
      polling_interval: 0.1
    - queues: default, mailers
      threads: 5
      processes: 2
      polling_interval: 0.1
    - queues: low_priority
      threads: 2
      processes: 1
      polling_interval: 1
```

### Concurrency Controls

```ruby
class ProcessOrderJob < ApplicationJob
  limits_concurrency to: 1, key: ->(order_id) { order_id }

  def perform(order_id)
    # Only one instance per order_id at a time
  end
end
```

## Common Pitfalls
1. **Setting threads too high** — Each thread holds a database connection. Match to your connection pool.
2. **Polling interval too aggressive** — 0.01s creates heavy DB load. Use 0.1s for most queues.

## Best Practices
1. **Scale processes, not threads** — Better fault isolation.
2. **Use environment variables for tuning** — Adjust without code changes.

## Summary
- Configure via `config/queue.yml` with dispatchers and workers.
- Configure separate workers for different queue priorities.
- Use `limits_concurrency` to prevent race conditions.
- Match thread count to database connection pool size.

## Code Examples

**Production queue.yml with dedicated workers for critical jobs**

```yaml
# config/queue.yml
production:
  dispatchers:
    - polling_interval: 1
      batch_size: 500
  workers:
    - queues: critical
      threads: 3
      processes: 1
    - queues: "*"
      threads: 5
      processes: 2
```


## Resources

- [Solid Queue Configuration](https://github.com/rails/solid_queue#configuration) — Configuration reference

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*