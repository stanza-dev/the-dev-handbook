---
source_course: "rails-background-processing"
source_lesson: "rails-background-jobs-queue-management"
---

# Queue Management

Proper queue design ensures jobs process efficiently.

## Queue Priority Design

```yaml
# config/sidekiq.yml
:queues:
  - critical      # Payment processing, security alerts
  - default       # Standard jobs
  - mailers       # Email delivery
  - images        # Image processing
  - reports       # Report generation (slow)
  - low           # Non-urgent tasks
```

## Queue-Specific Workers

Run separate processes for different queues:

```bash
# Critical jobs only (high priority)
bundle exec sidekiq -q critical -c 5

# Everything except low priority
bundle exec sidekiq -q critical -q default -q mailers -c 10

# Low priority batch processing
bundle exec sidekiq -q low -q reports -c 3
```

## Job-Specific Queues

```ruby
class PaymentJob < ApplicationJob
  queue_as :critical
  
  sidekiq_options retry: 5, backtrace: 10
end

class ReportJob < ApplicationJob
  queue_as :reports
  
  sidekiq_options retry: 3, dead: false
end
```

## Queue Monitoring

```ruby
# Check queue sizes
Sidekiq::Queue.new('default').size
Sidekiq::Queue.new('critical').latency  # Time oldest job waited

# Check scheduled jobs
Sidekiq::ScheduledSet.new.size

# Check retries
Sidekiq::RetrySet.new.size

# Check dead jobs
Sidekiq::DeadSet.new.size
```

## Pausing Queues

```ruby
# Pause a queue (jobs still enqueue but don't process)
Sidekiq::Queue.new('reports').pause!

# Resume
Sidekiq::Queue.new('reports').unpause!

# Check status
Sidekiq::Queue.new('reports').paused?
```

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-processing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*