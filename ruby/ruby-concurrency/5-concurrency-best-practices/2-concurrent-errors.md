---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-concurrent-errors"
---

# Error Handling in Concurrent Code

## Introduction
Exceptions in concurrent code behave differently than in sequential programs. A raised exception in a thread, fiber, or ractor does not automatically crash your program — it might vanish silently, propagate unexpectedly, or leave resources in a broken state. Understanding these differences is essential for building reliable concurrent systems.

## Key Concepts
- **Exception propagation**: How and when an exception raised in a concurrent context becomes visible to the caller.
- **Supervision**: A pattern where a parent process monitors child workers and restarts them on failure.
- **Graceful shutdown**: Stopping concurrent workers cleanly so that in-flight work finishes and resources are released.
- **Ensure block**: Ruby's `ensure` keyword guarantees cleanup code runs whether or not an exception occurred.

## Real World Context
In a background job processor, a single unhandled exception in a worker thread can silently corrupt a job queue, leaving tasks permanently stuck. In production, every concurrent worker must have explicit error handling, logging, and cleanup — otherwise failures become invisible and debugging turns into guesswork.

## Deep Dive

### Thread Exception Behavior

By default, Ruby threads swallow exceptions. The exception is stored on the thread object but never surfaces unless you explicitly check:

```ruby
thread = Thread.new do
  raise "Something broke!"
end

# The main thread continues as if nothing happened
sleep 0.1
puts "Main thread is fine"  # This prints!

# To see the error, call join or value:
thread.join   # Re-raises "Something broke!" in the main thread
```

After calling `join`, the stored exception propagates to the caller. Alternatively, `thread.value` both waits for completion and re-raises any exception.

You can change the default behavior globally:

```ruby
# Make ALL thread exceptions abort the program
Thread.abort_on_exception = true

# Or per-thread:
thread = Thread.new { raise "boom" }
thread.abort_on_exception = true
```

This is useful during development but risky in production where you want controlled error handling.

### Ractor Exception Behavior

Ractors propagate exceptions through the port system. When a Ractor raises, the exception is delivered when you call `value` or `receive` on it:

```ruby
ractor = Ractor.new do
  raise "Ractor failed!"
end

begin
  ractor.value
rescue Ractor::RemoteError => e
  puts e.message   # => "Ractor failed!"
  puts e.cause     # => #<RuntimeError: Ractor failed!>
end
```

Notice that the exception is wrapped in `Ractor::RemoteError`. The original exception is accessible via the `cause` method. This wrapping exists because the exception object must cross the Ractor isolation boundary.

### Fiber Exception Behavior

Fibers propagate exceptions directly to the caller of `resume`:

```ruby
fiber = Fiber.new do
  raise "Fiber error!"
end

begin
  fiber.resume
rescue RuntimeError => e
  puts "Caught: #{e.message}"  # => "Caught: Fiber error!"
end
```

This is the most intuitive model — exceptions propagate exactly as if the fiber's code ran inline at the point of `resume`.

### Supervision Pattern

For long-running workers, wrap each unit of work in its own error handler so one failure does not kill the entire worker:

```ruby
def supervised_worker(name, queue)
  Thread.new do
    loop do
      job = queue.pop
      break if job == :shutdown

      begin
        process_job(job)
      rescue StandardError => e
        logger.error("Worker #{name} failed on job #{job.id}: #{e.message}")
        logger.error(e.backtrace.first(5).join("\n"))
        # Worker continues processing the next job
      end
    end
  end
end
```

The `rescue` inside the loop catches errors for individual jobs while the worker thread stays alive to process more work.

### Cleanup with ensure

Always use `ensure` to release resources in concurrent code. Without it, an exception can leave file handles, database connections, or locks dangling:

```ruby
def fetch_with_cleanup(url)
  connection = open_connection(url)
  connection.get("/data")
rescue IOError => e
  logger.error("Connection failed: #{e.message}")
  nil
ensure
  connection&.close  # Always runs, even if an exception occurred
end
```

The `ensure` block runs regardless of whether the method succeeds, raises, or is interrupted. The safe navigation operator `&.` handles the case where `connection` was never assigned.

## Common Pitfalls
1. **Forgetting to join threads** — If you never call `join` or `value`, thread exceptions are silently lost. Always join threads you care about, or set `abort_on_exception` during development.
2. **Rescuing Exception instead of StandardError** — Catching `Exception` swallows `Interrupt` and `SystemExit`, making your program impossible to kill with Ctrl+C. Always rescue `StandardError` unless you have a specific reason.
3. **Missing ensure in locked sections** — If an exception occurs while a Mutex is held and you do not release it in `ensure`, other threads deadlock waiting for the lock forever.

## Best Practices
1. **Log before rescuing** — Always log the exception message and backtrace before swallowing an error. Silent failures are the hardest bugs to diagnose.
2. **Use structured error handling per worker** — Wrap each job or request in its own begin/rescue/ensure so one failure does not take down the whole pool.
3. **Propagate fatal errors upward** — Re-raise exceptions that indicate unrecoverable conditions (out of memory, configuration errors) rather than retrying endlessly.

## Summary
- Threads swallow exceptions by default; always call `join` or `value` to surface them.
- Ractor exceptions are wrapped in `Ractor::RemoteError` and delivered through `value`.
- Fiber exceptions propagate directly to the caller of `resume`.
- Use the supervision pattern to keep workers alive after individual failures.
- Always use `ensure` for resource cleanup in concurrent code.

## Code Examples

**A supervised thread pool where each job is wrapped in error handling — one failing job never kills the worker thread**

```ruby
# Supervised thread pool with per-job error handling
class SupervisedPool
  def initialize(size, logger:)
    @queue = Thread::Queue.new
    @logger = logger
    @threads = size.times.map { |i| spawn_worker(i) }
  end

  def schedule(&block)
    @queue << block
  end

  def shutdown
    @threads.size.times { @queue << :stop }
    @threads.each(&:join)
  end

  private

  def spawn_worker(id)
    Thread.new do
      while (job = @queue.pop) != :stop
        begin
          job.call
        rescue StandardError => e
          @logger.error("Worker #{id}: #{e.message}")
        end
      end
    end
  end
end

# Usage:
# pool = SupervisedPool.new(4, logger: Logger.new($stdout))
# pool.schedule { process_order(order) }
# pool.shutdown
```


## Resources

- [Ractor::RemoteError Documentation](https://docs.ruby-lang.org/en/4.0/Ractor/RemoteError.html) — Reference for the exception wrapper used when Ractor errors cross isolation boundaries

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*