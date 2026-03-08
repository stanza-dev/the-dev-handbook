---
source_course: "rails-background-jobs"
source_lesson: "rails-background-jobs-job-monitoring"
---

# Job Monitoring

## Introduction
Background jobs run without user-facing feedback, making monitoring essential. Without visibility into queue depth, latency, and failure rates, problems go undetected.

## Key Concepts
- **Queue Depth**: Jobs waiting to be processed.
- **Queue Latency**: Time between enqueue and execution start.
- **Failure Rate**: Percentage of jobs that fail.

## Real World Context
At 2 AM, your payment gateway starts timing out. Without monitoring, failed jobs pile up silently. With monitoring, an alert fires immediately.

## Deep Dive

### Solid Queue Monitoring

```ruby
SolidQueue::ReadyExecution.count
SolidQueue::ScheduledExecution.count
SolidQueue::FailedExecution.count
SolidQueue::ReadyExecution.where(queue_name: "default").count
```

### Health Check Endpoint

```ruby
class HealthController < ApplicationController
  def jobs
    ready = SolidQueue::ReadyExecution.count
    failed = SolidQueue::FailedExecution.count
    healthy = failed < 100 && ready < 10_000
    render json: { healthy: healthy, ready: ready, failed: failed },
           status: healthy ? :ok : :service_unavailable
  end
end
```

### ActiveSupport Notifications

```ruby
ActiveSupport::Notifications.subscribe("perform.active_job") do |*args|
  event = ActiveSupport::Notifications::Event.new(*args)
  job = event.payload[:job]
  Rails.logger.info "Job: #{job.class.name} queue=#{job.queue_name} duration=#{event.duration.round(1)}ms"
end
```

## Common Pitfalls
1. **Only monitoring failures** — A queue with zero failures but 10-minute latency is still broken.
2. **Alert thresholds too low** — Normal spikes (deploys) should not page your team.

## Best Practices
1. **Create a /health/jobs endpoint** for external monitoring.
2. **Subscribe to Active Job notifications** for standardized metrics.

## Summary
- Monitor queue depth, latency, and failure rates.
- Query SolidQueue tables or use ActiveSupport notifications.
- Create health check endpoints for external monitoring.
- Alert on sustained high latency and rising failure counts.

## Code Examples

**ActiveSupport::Notifications for job metrics — works with any backend**

```ruby
ActiveSupport::Notifications.subscribe("perform.active_job") do |*args|
  event = ActiveSupport::Notifications::Event.new(*args)
  job = event.payload[:job]
  StatsD.timing("jobs.duration", event.duration, tags: ["class:#{job.class.name}"])
end
```


## Resources

- [Active Support Instrumentation](https://guides.rubyonrails.org/active_support_instrumentation.html#active-job) — Active Job instrumentation events

---

> 📘 *This lesson is part of the [Background Processing in Rails](https://stanza.dev/courses/rails-background-jobs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*