---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-application-monitoring"
---

# Application Performance Monitoring

## Introduction
APM tools provide continuous visibility into production performance — tracking response times, error rates, and throughput over time. While profiling is for finding specific bottlenecks, monitoring is for detecting when things go wrong.

## Key Concepts
- **APM (Application Performance Monitoring)**: Tools like New Relic or Scout that track production performance metrics continuously.
- **Percentiles (p50, p95, p99)**: Response time metrics showing what percentage of requests complete within that time. p99 = 99% of requests are faster than this value.
- **Apdex Score**: A standardized metric (0-1) measuring user satisfaction based on response time thresholds.

## Real World Context
Your average response time might be 200ms, but p99 could be 8 seconds. Without monitoring percentiles, a bad experience for 1% of users — potentially thousands per day — goes undetected.

## Deep Dive

### Key Metrics to Track

- **Response Time**: p50, p95, p99 percentiles
- **Throughput**: Requests per minute
- **Error Rate**: Percentage of 5xx responses
- **Database Time**: Time spent in SQL queries
- **Queue Latency**: How long background jobs wait before processing

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

### Health Check Endpoint

```ruby
class HealthController < ApplicationController
  skip_before_action :authenticate_user!

  def show
    checks = {
      database: check_database,
      queue: check_solid_queue
    }
    status = checks.values.all? ? :ok : :service_unavailable
    render json: checks, status: status
  end

  private

  def check_database
    ActiveRecord::Base.connection.execute('SELECT 1')
    true
  rescue StandardError
    false
  end

  def check_solid_queue
    SolidQueue::Job.where(finished_at: nil).count
    true
  rescue StandardError
    false
  end
end
```

### Custom Metrics with StatsD

```ruby
class OrdersController < ApplicationController
  def create
    start_time = Time.current
    @order = Order.create!(order_params)

    StatsD.increment('orders.created')
    StatsD.timing('orders.create_time', Time.current - start_time)
  rescue => e
    StatsD.increment('orders.failed')
    raise
  end
end
```

## Common Pitfalls
1. **Only tracking averages** — Averages hide outliers. Track p95 and p99 to catch performance issues affecting a minority of users.
2. **Not monitoring queue latency** — A healthy web server with a backed-up job queue means emails aren't sending and imports aren't processing.

## Best Practices
1. **Set up alerts on p95 response time** — Alert when p95 exceeds your SLA threshold (e.g., 2 seconds).
2. **Monitor database connection pool** — Track active and waiting connections to detect pool exhaustion before it causes errors.

## Summary
- APM tools provide continuous production performance visibility.
- Track percentiles (p50, p95, p99), not just averages.
- Monitor both web response times and background job queue latency.
- Set up health check endpoints for load balancers and uptime monitoring.

## Code Examples

**A minimal health check endpoint — load balancers use this to verify the app can serve requests**

```ruby
# Health check endpoint for load balancers
class HealthController < ApplicationController
  skip_before_action :authenticate_user!

  def show
    ActiveRecord::Base.connection.execute('SELECT 1')
    head :ok
  rescue StandardError
    head :service_unavailable
  end
end

# config/routes.rb
get '/health', to: 'health#show'
```


## Resources

- [Rails Instrumentation Guide](https://guides.rubyonrails.org/active_support_instrumentation.html) — Official Rails guide on built-in instrumentation events for monitoring

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*