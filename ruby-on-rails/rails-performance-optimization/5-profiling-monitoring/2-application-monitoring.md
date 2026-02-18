---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-application-monitoring"
---

# Application Monitoring

APM tools provide ongoing visibility into production performance.

## Key Metrics to Monitor

- **Response Time**: p50, p95, p99 percentiles
- **Throughput**: Requests per minute
- **Error Rate**: Percentage of failed requests
- **Apdex Score**: User satisfaction metric
- **Database Time**: Time spent in queries

## Popular APM Solutions

### New Relic

```ruby
# Gemfile
gem 'newrelic_rpm'

# config/newrelic.yml
production:
  license_key: <%= ENV['NEW_RELIC_LICENSE_KEY'] %>
  app_name: My Rails App
```

### Scout APM

```ruby
# Gemfile
gem 'scout_apm'

# config/scout_apm.yml
production:
  key: <%= ENV['SCOUT_KEY'] %>
  name: My Rails App
```

## Custom Metrics

```ruby
# With StatsD
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

## Health Checks

```ruby
# config/routes.rb
get '/health', to: 'health#show'

# app/controllers/health_controller.rb
class HealthController < ApplicationController
  skip_before_action :authenticate_user!
  
  def show
    checks = {
      database: database_connected?,
      redis: redis_connected?,
      sidekiq: sidekiq_running?
    }
    
    status = checks.values.all? ? :ok : :service_unavailable
    render json: checks, status: status
  end
  
  private
  
  def database_connected?
    ActiveRecord::Base.connection.execute('SELECT 1')
    true
  rescue
    false
  end
  
  def redis_connected?
    Redis.current.ping == 'PONG'
  rescue
    false
  end
end
```

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*