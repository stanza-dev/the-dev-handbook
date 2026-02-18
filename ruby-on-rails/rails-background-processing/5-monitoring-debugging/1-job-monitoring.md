---
source_course: "rails-background-processing"
source_lesson: "rails-background-jobs-job-monitoring"
---

# Job Monitoring

Monitoring ensures your background jobs run smoothly.

## Sidekiq Metrics

```ruby
# Get current stats
stats = Sidekiq::Stats.new
stats.processed      # Total jobs processed
stats.failed         # Total failures
stats.enqueued       # Jobs waiting
stats.scheduled_size # Scheduled jobs
stats.retry_size     # Jobs to retry
stats.dead_size      # Dead jobs

# Queue-specific
queue = Sidekiq::Queue.new('default')
queue.size           # Jobs in queue
queue.latency        # Wait time (seconds)
```

## Custom Metrics

```ruby
# config/initializers/sidekiq.rb
Sidekiq.configure_server do |config|
  config.server_middleware do |chain|
    chain.add JobMetricsMiddleware
  end
end

class JobMetricsMiddleware
  def call(worker, job, queue)
    start_time = Time.current
    
    begin
      yield
      StatsD.increment('jobs.completed', tags: ["class:#{job['class']}", "queue:#{queue}"])
    rescue => e
      StatsD.increment('jobs.failed', tags: ["class:#{job['class']}", "queue:#{queue}"])
      raise e
    ensure
      duration = Time.current - start_time
      StatsD.timing('jobs.duration', duration * 1000, tags: ["class:#{job['class']}"])
    end
  end
end
```

## Health Checks

```ruby
# app/controllers/health_controller.rb
class HealthController < ApplicationController
  def sidekiq
    stats = Sidekiq::Stats.new
    queues = Sidekiq::Queue.all
    
    unhealthy_queues = queues.select { |q| q.latency > 300 }  # 5 min threshold
    
    status = {
      healthy: unhealthy_queues.empty?,
      processed: stats.processed,
      failed: stats.failed,
      enqueued: stats.enqueued,
      retry_size: stats.retry_size,
      queues: queues.map { |q| { name: q.name, size: q.size, latency: q.latency } }
    }
    
    render json: status, status: status[:healthy] ? :ok : :service_unavailable
  end
end
```

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-processing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*