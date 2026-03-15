---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-fiber-scheduler-interface"
---

# The Fiber Scheduler Interface

## Introduction
Ruby 3.0 introduced the Fiber Scheduler interface, and Ruby 4.0 enhances it with new hooks like `fiber_interrupt`, `yield`, and `io_close`. When a scheduler is set, Ruby automatically pauses fibers during blocking I/O and lets other fibers run — turning synchronous-looking code into concurrent code without callbacks or async/await syntax.

## Key Concepts
- **Fiber Scheduler**: An object that implements the `Fiber::Scheduler` interface, intercepting blocking operations (I/O, sleep, DNS) and multiplexing fibers onto a single thread.
- **Fiber.schedule**: Creates and schedules a fiber within the current scheduler's event loop.
- **Transparent async**: Standard library methods (Net::HTTP, IO.read, sleep) automatically become non-blocking when a scheduler is active.

## Real World Context
The Falcon web server uses `async` (which implements Fiber::Scheduler) to handle 10,000+ concurrent connections on a single thread. Unlike Node.js callbacks or Go goroutines, Ruby's approach lets you write plain synchronous code that the scheduler makes concurrent. No refactoring needed — your existing code just works.

## Deep Dive

### How the Scheduler Works

When you set a fiber scheduler, Ruby intercepts every blocking operation. Instead of blocking the entire thread, it yields the current fiber and lets another fiber run.

```ruby
# Without scheduler: each request blocks the thread sequentially
# With scheduler: fibers yield during I/O, allowing concurrent progress

Fiber.set_scheduler(MyScheduler.new)

Fiber.schedule do
  # This looks synchronous but yields during the HTTP call
  response = Net::HTTP.get(URI("https://api.example.com/users"))
  puts "Users: #{response.length} bytes"
end

Fiber.schedule do
  # This fiber runs while the first is waiting for HTTP response
  response = Net::HTTP.get(URI("https://api.example.com/orders"))
  puts "Orders: #{response.length} bytes"
end

# The scheduler runs both fibers concurrently on one thread
```

Both HTTP requests happen concurrently. While one fiber waits for a network response, the scheduler runs the other fiber.

### Ruby 4.0 Scheduler Enhancements

Ruby 4.0 adds several new hooks to the scheduler interface, giving implementers finer-grained control.

```ruby
class EnhancedScheduler
  # Interrupt a fiber that is waiting (new in Ruby 4.0)
  def fiber_interrupt(fiber, exception)
    # Wake the fiber and raise the exception inside it
    # Useful for timeouts and cancellation
  end

  # Explicit yield point (new in Ruby 4.0)
  def yield
    # Allows the scheduler to run other ready fibers
    # Called when a fiber wants to voluntarily give up control
  end

  # I/O close notification (new in Ruby 4.0)
  def io_close(io)
    # Clean up scheduler state when an IO object is closed
    # Receives a numeric file descriptor, not an IO object
  end

  # I/O write hook (changed in Ruby 4.0 — now invoked on buffer flush)
  def io_write(io, buffer, length, offset)
    # Called on buffer flush, not only on explicit writes
  end

  # Standard hooks
  def io_wait(io, events, timeout) = # ...
  def kernel_sleep(duration) = # ...
  def block(blocker, timeout = nil) = # ...
  def unblock(blocker, fiber) = # ...
  def close = # ...
end
```

The `fiber_interrupt` hook is particularly useful: it lets you cancel a fiber that is blocked on I/O, enabling timeout patterns without threads.

### Using the async Gem

In practice, you rarely implement Fiber::Scheduler yourself. The `async` gem provides a battle-tested implementation.

```ruby
require 'async'
require 'async/http/internet'

Async do |task|
  internet = Async::HTTP::Internet.new

  urls = [
    "https://api.example.com/users",
    "https://api.example.com/orders",
    "https://api.example.com/products"
  ]

  # All three requests run concurrently on one thread
  tasks = urls.map do |url|
    task.async { internet.get(url) }
  end

  responses = tasks.map(&:wait)
  puts "Fetched #{responses.length} endpoints concurrently"
ensure
  internet.close
end
```

The `Async` block sets up a scheduler. Each `task.async` creates a fiber. The three HTTP requests execute concurrently, and `wait` collects results.

## Common Pitfalls
1. **Expecting parallelism** — Fiber schedulers provide concurrency (interleaved execution), not parallelism (simultaneous execution). CPU-bound work in one fiber blocks all other fibers on the same thread.
2. **Using blocking libraries without scheduler support** — Not all gems release the GVL or support the scheduler interface. Legacy C extension gems may still block the entire thread. Check gem documentation for async compatibility.

## Best Practices
1. **Use `async` gem for new I/O-heavy projects** — It handles all scheduler internals correctly and provides async-compatible HTTP, DNS, and socket libraries out of the box.
2. **Design around `Fiber::Scheduler#yield`** — In CPU-intensive loops within a scheduled fiber, call `Fiber.scheduler.yield` periodically to let other fibers run and prevent starvation.

## Summary
- The Fiber Scheduler intercepts blocking I/O and automatically yields fibers, enabling concurrent execution on a single thread.
- Ruby 4.0 adds `fiber_interrupt` for cancellation, `yield` for explicit yield points, `io_close` for resource cleanup, and changes `io_write` to fire on buffer flush.
- The `async` gem is the production-standard Fiber::Scheduler implementation.
- Sync-looking code becomes concurrent without callbacks or refactoring.

## Code Examples

**Three log files read concurrently using async — each fiber yields during disk I/O while others continue**

```ruby
require 'async'

# Concurrent file reads using the async scheduler
Async do |task|
  paths = ["/var/log/app.log", "/var/log/error.log", "/var/log/access.log"]

  reads = paths.map do |path|
    task.async { File.read(path) }  # Yields during disk I/O
  end

  contents = reads.map(&:wait)
  total_lines = contents.sum { |c| c.lines.count }
  puts "Read #{total_lines} total lines from #{paths.length} files"
end
```


## Resources

- [Fiber::Scheduler Interface](https://docs.ruby-lang.org/en/4.0/Fiber/Scheduler.html) — Official Fiber::Scheduler interface documentation covering all hooks a scheduler must implement

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*