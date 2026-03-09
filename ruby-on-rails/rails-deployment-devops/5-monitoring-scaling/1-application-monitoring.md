---
source_course: "rails-deployment-devops"
source_lesson: "rails-deployment-application-monitoring"
---

# Application Performance Monitoring

## Introduction
Monitoring provides continuous visibility into application health. While profiling is for diagnosing specific problems, APM tools track performance over time, revealing trends and catching regressions.

## Key Concepts
- **APM (Application Performance Monitoring)**: Tools like New Relic or Scout that continuously track response times, error rates, and throughput in production.
- **Percentiles**: Response time metrics (p50, p95, p99) that show the distribution of request speeds, revealing outliers that averages hide.
- **Instrumentation**: Rails' built-in event system that fires notifications for SQL queries, controller actions, and view rendering.

## Real World Context
Your average response time is 200ms, but monitoring reveals p99 is 12 seconds. 1% of users — potentially hundreds per day — experience terrible performance. Without percentile monitoring, this goes undetected.

## Deep Dive

### Rails Built-in Instrumentation

```ruby
ActiveSupport::Notifications.subscribe('process_action.action_controller') do |*args|
  event = ActiveSupport::Notifications::Event.new(*args)
  Rails.logger.info({
    path: event.payload[:path],
    status: event.payload[:status],
    duration: event.duration,
    db_time: event.payload[:db_runtime]
  }.to_json)
end
```

### New Relic Setup

```ruby
# Gemfile
gem 'newrelic_rpm'
```

```yaml
# config/newrelic.yml
production:
  license_key: <%= ENV['NEW_RELIC_LICENSE_KEY'] %>
  app_name: My Rails App
```

### Custom Metrics with StatsD

```ruby
class OrdersController < ApplicationController
  def create
    start_time = Time.current
    @order = Order.create!(order_params)
    StatsD.increment('orders.created')
    StatsD.timing('orders.create_time', (Time.current - start_time) * 1000)
  rescue => e
    StatsD.increment('orders.failed')
    raise
  end
end
```

## Common Pitfalls
1. **Only tracking averages** — Averages hide outliers. Track p95 and p99 to catch the worst user experiences.
2. **Not monitoring Solid Queue latency** — A healthy web server with a backed-up job queue means emails aren't sending and imports aren't processing.

## Best Practices
1. **Set alerts on p95 response time** — Alert when response times exceed your SLA threshold.
2. **Monitor error rates** — A sudden spike in 500 errors after a deploy indicates a problem. Set alerts at > 1% error rate.

## Summary
- APM tools provide continuous production monitoring.
- Track percentiles (p50, p95, p99), not just averages.
- Use Rails' built-in instrumentation for custom event tracking.
- Monitor both web response times and job queue latency.

## Code Examples

**Subscribing to Rails events to automatically log slow SQL queries over 100ms**

```ruby
# Log slow SQL queries automatically
ActiveSupport::Notifications.subscribe('sql.active_record') do |*args|
  event = ActiveSupport::Notifications::Event.new(*args)
  if event.duration > 100
    Rails.logger.warn "SLOW QUERY (#{event.duration.round}ms): #{event.payload[:sql]}"
  end
end
```


## Resources

- [Active Support Instrumentation](https://guides.rubyonrails.org/active_support_instrumentation.html) — Official Rails guide on built-in instrumentation events

---

> 📘 *This lesson is part of the [Rails Deployment and DevOps](https://stanza.dev/courses/rails-deployment-devops) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*