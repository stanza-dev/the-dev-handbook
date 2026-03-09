---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-rails-profiling-tools"
---

# Rails Profiling Tools

## Introduction
Profiling tools reveal where your application spends time and memory. Without profiling, optimization is guesswork. Rails provides built-in instrumentation and integrates well with third-party profiling gems.

## Key Concepts
- **rack-mini-profiler**: A gem that displays a timing badge on every page showing request duration, SQL queries, and memory usage.
- **ActiveSupport::Notifications**: Rails' built-in instrumentation system that fires events for SQL queries, controller actions, view rendering, and more.
- **Benchmark**: Ruby's standard library for measuring execution time of code blocks.

## Real World Context
A slow page might feel like a view rendering problem, but rack-mini-profiler reveals that 80% of the time is spent in 47 SQL queries. Without profiling, you'd waste time optimizing the wrong thing.

## Deep Dive

### Rack Mini Profiler

```ruby
# Gemfile
gem 'rack-mini-profiler'
gem 'memory_profiler'  # For memory profiling
gem 'stackprof'        # For CPU profiling
```

Shows a timing badge on every page with total request time, SQL queries and times, and memory allocation.

### Benchmark Module

```ruby
require 'benchmark'

Benchmark.bm do |x|
  x.report('includes:') { Post.includes(:author).to_a }
  x.report('preload:')  { Post.preload(:author).to_a }
end

# In Rails console
Benchmark.measure { User.all.to_a }
```

### ActiveSupport::Notifications

```ruby
# config/initializers/instrumentation.rb
ActiveSupport::Notifications.subscribe('sql.active_record') do |*args|
  event = ActiveSupport::Notifications::Event.new(*args)
  if event.duration > 100
    Rails.logger.warn "Slow query (#{event.duration}ms): #{event.payload[:sql]}"
  end
end

ActiveSupport::Notifications.subscribe('process_action.action_controller') do |*args|
  event = ActiveSupport::Notifications::Event.new(*args)
  Rails.logger.info "Request: #{event.payload[:path]} took #{event.duration}ms"
end
```

### Custom Instrumentation

```ruby
class OrderService
  def process(order)
    ActiveSupport::Notifications.instrument('process.order_service', order_id: order.id) do
      # Processing logic
    end
  end
end
```

## Common Pitfalls
1. **Profiling only in development** — Development and production have vastly different data sizes and configurations. Profile with production-like data.
2. **Optimizing without measuring** — Always measure before and after changes. A "optimization" that doesn't show improvement in profiling was wasted effort.

## Best Practices
1. **Use rack-mini-profiler in development** — It gives instant feedback on every page load, catching regressions as you develop.
2. **Subscribe to sql.active_record in staging** — Log all queries over 100ms to identify slow queries before they hit production.

## Summary
- rack-mini-profiler shows timing, SQL, and memory on every page.
- ActiveSupport::Notifications lets you subscribe to Rails internal events.
- Use Benchmark for comparing alternative implementations.
- Always measure before and after optimizing — don't guess.

## Code Examples

**Custom instrumentation that logs any SQL query taking over 100ms — catches slow queries automatically**

```ruby
# Log slow SQL queries automatically
ActiveSupport::Notifications.subscribe('sql.active_record') do |*args|
  event = ActiveSupport::Notifications::Event.new(*args)
  if event.duration > 100  # milliseconds
    Rails.logger.warn(
      "SLOW QUERY (#{event.duration.round(1)}ms): #{event.payload[:sql]}"
    )
  end
end
```


## Resources

- [Active Support Instrumentation](https://guides.rubyonrails.org/active_support_instrumentation.html) — Official Rails guide on ActiveSupport::Notifications and built-in instrumentation hooks

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*