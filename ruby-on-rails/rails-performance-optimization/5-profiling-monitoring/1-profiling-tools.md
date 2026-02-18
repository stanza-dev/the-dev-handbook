---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-rails-profiling-tools"
---

# Rails Profiling Tools

Profiling helps identify where your application spends time.

## Rack Mini Profiler

Add to Gemfile:
```ruby
gem 'rack-mini-profiler'
gem 'memory_profiler'  # For memory profiling
gem 'stackprof'        # For CPU profiling
```

Shows timing badge on every page with:
- Total request time
- SQL queries and times
- Memory allocation

## Rails Panel (Chrome Extension)

View Rails logs in Chrome DevTools:
- Request parameters
- SQL queries
- Rendering times
- Cache hits/misses

## Benchmark Module

```ruby
require 'benchmark'

Benchmark.bm do |x|
  x.report('Method A:') { 1000.times { method_a } }
  x.report('Method B:') { 1000.times { method_b } }
end

# In Rails console
Benchmark.measure { User.all.to_a }
```

## ActiveSupport::Notifications

Subscribe to Rails events:

```ruby
# config/initializers/instrumentation.rb
ActiveSupport::Notifications.subscribe('sql.active_record') do |*args|
  event = ActiveSupport::Notifications::Event.new(*args)
  
  if event.duration > 100  # Log slow queries
    Rails.logger.warn "Slow query (#{event.duration}ms): #{event.payload[:sql]}"
  end
end

ActiveSupport::Notifications.subscribe('process_action.action_controller') do |*args|
  event = ActiveSupport::Notifications::Event.new(*args)
  
  Rails.logger.info "Request: #{event.payload[:path]} took #{event.duration}ms"
end
```

## Custom Instrumentation

```ruby
class OrderService
  def process(order)
    ActiveSupport::Notifications.instrument('process.order_service', order_id: order.id) do
      # Processing logic
    end
  end
end
```

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*