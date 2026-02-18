---
source_course: "rails-deployment-devops"
source_lesson: "rails-deployment-application-monitoring"
---

# Application Monitoring

Monitoring provides visibility into application health and performance.

## Key Metrics to Track

- **Response Time**: p50, p95, p99 percentiles
- **Throughput**: Requests per minute/second
- **Error Rate**: Percentage of failed requests
- **Apdex Score**: User satisfaction metric (0-1)
- **Database Time**: Time spent in queries
- **Queue Latency**: Background job wait times

## Rails Built-in Instrumentation

```ruby
# Subscribe to Rails events
ActiveSupport::Notifications.subscribe('process_action.action_controller') do |*args|
  event = ActiveSupport::Notifications::Event.new(*args)
  
  Rails.logger.info({
    path: event.payload[:path],
    method: event.payload[:method],
    status: event.payload[:status],
    duration: event.duration,
    db_time: event.payload[:db_runtime],
    view_time: event.payload[:view_runtime]
  }.to_json)
end

# SQL query monitoring
ActiveSupport::Notifications.subscribe('sql.active_record') do |*args|
  event = ActiveSupport::Notifications::Event.new(*args)
  
  if event.duration > 100  # Log slow queries
    Rails.logger.warn "Slow query (#{event.duration}ms): #{event.payload[:sql]}"
  end
end
```

## APM Solutions

### New Relic

```ruby
# Gemfile
gem 'newrelic_rpm'
```

```yaml
# config/newrelic.yml
production:
  license_key: <%= ENV['NEW_RELIC_LICENSE_KEY'] %>
  app_name: My Rails App
  log_level: info
```

### Scout APM

```ruby
# Gemfile
gem 'scout_apm'
```

```yaml
# config/scout_apm.yml
production:
  key: <%= ENV['SCOUT_KEY'] %>
  name: My Rails App
  monitor: true
```

## Custom Metrics with StatsD

```ruby
# config/initializers/statsd.rb
STATSD = Statsd.new(ENV['STATSD_HOST'], 8125)

# Usage in application
class OrdersController < ApplicationController
  def create
    start_time = Time.current
    @order = Order.create!(order_params)
    
    STATSD.increment('orders.created')
    STATSD.timing('orders.create_time', (Time.current - start_time) * 1000)
  rescue => e
    STATSD.increment('orders.failed')
    raise
  end
end
```

---

> 📘 *This lesson is part of the [Rails Deployment and DevOps](https://stanza.dev/courses/rails-deployment-devops) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*