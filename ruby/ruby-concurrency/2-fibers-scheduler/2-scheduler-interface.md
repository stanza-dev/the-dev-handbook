---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-fiber-scheduler-interface"
---

# Non-Blocking I/O with Fiber Scheduler

Ruby 3+ introduced the Fiber Scheduler interface. When you set a scheduler, Ruby automatically pauses fibers during blocking I/O.

## How It Works

```ruby
# Without scheduler: blocking I/O stops everything
# With scheduler: fiber yields, another fiber runs

Fiber.set_scheduler(MyScheduler.new)

Fiber.schedule do
  # This blocks normally...
  result = Net::HTTP.get(URI("https://example.com"))
  # But with a scheduler, it yields and other fibers run!
end
```

## Ruby 4.0 Scheduler Improvements

Ruby 4.0 adds new methods to `Fiber::Scheduler`:

```ruby
class MyScheduler
  # New in Ruby 4.0
  def fiber_interrupt(fiber, exception)
    # Handle fiber interruption
  end

  def yield
    # Explicit yield point
  end
end
```

## Using the async Gem

```ruby
require 'async'

Async do |task|
  # These run concurrently!
  task.async { fetch_url("https://api1.example.com") }
  task.async { fetch_url("https://api2.example.com") }
  task.async { fetch_url("https://api3.example.com") }
end  # Waits for all to complete
```

## Benefits

- Sync-looking code that runs async
- No callback hell
- Very low overhead (thousands of fibers possible)
- Works with standard library I/O

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*