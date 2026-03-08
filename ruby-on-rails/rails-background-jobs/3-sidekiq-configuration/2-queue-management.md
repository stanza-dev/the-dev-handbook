---
source_course: "rails-background-jobs"
source_lesson: "rails-background-jobs-queue-management"
---

# Queue Management

## Introduction
Proper queue design ensures jobs process efficiently. Sidekiq lets you define multiple queues with different priorities and run separate worker processes for each.

## Key Concepts
- **Queue Priority**: Weighted polling — higher-weight queues are checked more frequently.
- **Queue-Specific Workers**: Running separate Sidekiq processes for different queues.
- **Queue Monitoring**: Checking queue sizes, latency, and retry counts.

## Real World Context
A production app might have: critical (payments), default (standard work), mailers (emails), low (reports). Each gets appropriate worker resources.

## Deep Dive

### Queue Priority Design

```yaml
# config/sidekiq.yml
:queues:
  - critical      # Payment processing, security
  - default       # Standard jobs
  - mailers       # Email delivery
  - low           # Non-urgent tasks
```

### Queue-Specific Workers

```bash
# Critical jobs only
bundle exec sidekiq -q critical -c 5

# Everything except low priority
bundle exec sidekiq -q critical -q default -q mailers -c 10

# Batch processing
bundle exec sidekiq -q low -q reports -c 3
```

### Job-Specific Queues

```ruby
class PaymentJob < ApplicationJob
  queue_as :critical
  sidekiq_options retry: 5, backtrace: 10
end
```

### Queue Monitoring

```ruby
Sidekiq::Queue.new('default').size
Sidekiq::Queue.new('critical').latency
Sidekiq::RetrySet.new.size
Sidekiq::DeadSet.new.size
```

## Common Pitfalls
1. **Using only one queue** — All jobs compete for the same workers. Separate by priority.
2. **Not monitoring queue latency** — A growing queue with increasing latency signals you need more workers.

## Best Practices
1. **Name queues by purpose** — `:critical`, `:mailers`, `:imports` are clearer than `:high`, `:medium`, `:low`.
2. **Run dedicated workers for critical queues** — Ensures time-sensitive jobs are never blocked.

## Summary
- Design queues by job priority and purpose.
- Use queue weights for relative priority.
- Run separate worker processes for different queue tiers.
- Monitor queue sizes and latency for health.

## Code Examples

**Sidekiq queue monitoring — check sizes, latency, and failure counts**

```ruby
# Check queue health
stats = Sidekiq::Stats.new
puts "Processed: #{stats.processed}"
puts "Failed: #{stats.failed}"
puts "Enqueued: #{stats.enqueued}"

Sidekiq::Queue.all.each do |q|
  puts "#{q.name}: #{q.size} jobs, #{q.latency.round(1)}s latency"
end
```


## Resources

- [Sidekiq Queues](https://github.com/sidekiq/sidekiq/wiki/Advanced-Options#queues) — Sidekiq queue configuration

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*