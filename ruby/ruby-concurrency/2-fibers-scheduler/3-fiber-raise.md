---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-fiber-raise"
---

# Fiber Error Handling

## Introduction
Fibers need robust error handling, especially in long-running schedulers and async servers. Ruby lets you raise exceptions inside a fiber from outside using `Fiber#raise`, and Ruby 4.0 enhances this with cause chaining. This lesson covers fiber error injection, cleanup patterns, and dead fiber detection.

## Key Concepts
- **Fiber#raise**: Injects an exception into a suspended fiber, resuming it at the point where it last yielded and raising the exception there.
- **Cause Chain (Ruby 4.0)**: The `cause:` keyword on `Fiber#raise` attaches an original exception as the cause, preserving the full error context.
- **Dead Fiber**: A fiber whose block has finished executing — calling `resume` or `raise` on it raises `FiberError`.

## Real World Context
In an async web server, you need to cancel fibers that exceed a timeout. When a client disconnects mid-request, the scheduler must interrupt the fiber handling that request. `Fiber#raise` provides the mechanism: inject a timeout or cancellation error, and the fiber's `ensure` block cleans up database connections, temp files, and other resources.

## Deep Dive

### Basic Fiber#raise

You can inject an exception into a paused fiber. The exception is raised at the point where the fiber last called `Fiber.yield`.

```ruby
long_running = Fiber.new do
  begin
    loop do
      puts "Working..."
      Fiber.yield
    end
  rescue => error
    puts "Interrupted: #{error.message}"
  end
end

long_running.resume  # Prints: Working...
long_running.resume  # Prints: Working...
long_running.raise(RuntimeError, "Timeout exceeded")
# Prints: Interrupted: Timeout exceeded
```

After `raise`, the fiber resumes at its last `Fiber.yield` point, but instead of continuing normally, the exception is raised there. The `rescue` block catches it.

### Ruby 4.0: raise with Cause Chain

Ruby 4.0 adds the `cause:` keyword to `Fiber#raise`, letting you attach an original exception as the cause. This preserves the full error chain for debugging.

```ruby
original_error = IOError.new("Connection reset by peer")

request_handler = Fiber.new do
  begin
    Fiber.yield  # Wait for work
  rescue => error
    puts "Error: #{error.message}"         # => "Request cancelled"
    puts "Cause: #{error.cause.message}"   # => "Connection reset by peer"
    puts "Cause class: #{error.cause.class}"  # => IOError
  end
end

request_handler.resume
request_handler.raise(
  RuntimeError, "Request cancelled",
  cause: original_error
)
```

The `cause:` parameter creates a linked chain. When logging or re-raising, the full history is preserved — you know that the request was cancelled because the underlying connection was reset.

### Cleanup with ensure

Fibers support `ensure` blocks just like regular methods. Whether the fiber completes normally, raises an exception, or is interrupted via `Fiber#raise`, the `ensure` block runs.

```ruby
db_worker = Fiber.new do
  connection = Database.connect("localhost:5432")
  begin
    loop do
      query = Fiber.yield
      connection.execute(query)
    end
  ensure
    connection.close  # Always runs, even on Fiber#raise
    puts "Database connection closed"
  end
end

db_worker.resume
db_worker.resume("SELECT * FROM users")
db_worker.raise(StopIteration)  # Triggers ensure, closes connection
```

The `ensure` block guarantees cleanup regardless of how the fiber terminates.

### Dead Fiber Detection

Once a fiber's block finishes, the fiber is dead. Attempting to resume or raise on it causes `FiberError`.

```ruby
short_fiber = Fiber.new { "done" }
short_fiber.resume   # => "done"

puts short_fiber.alive?  # => false

begin
  short_fiber.resume  # FiberError: dead fiber called
rescue FiberError => error
  puts error.message  # => "dead fiber called"
end
```

Always check `alive?` before resuming a fiber if the number of yields is not known at compile time.

## Common Pitfalls
1. **Raising on a dead fiber** — Calling `raise` on a fiber that has already finished throws `FiberError`, not the exception you wanted to raise. Always check `fiber.alive?` first.
2. **Forgetting ensure blocks** — If a fiber holds resources (connections, file handles, locks) and you interrupt it with `raise`, those resources leak unless an `ensure` block releases them.

## Best Practices
1. **Always wrap resource-holding fibers in begin/ensure** — Any fiber that opens connections, files, or acquires locks should have an `ensure` block that releases them. This is critical in scheduler-based async code.
2. **Use cause chains for error context** — When wrapping a low-level error in a higher-level one, pass the original as `cause:`. This makes debugging production errors much easier.

## Summary
- `Fiber#raise` injects an exception into a suspended fiber at its last yield point.
- Ruby 4.0's `cause:` keyword preserves the full error chain when wrapping exceptions.
- `ensure` blocks in fibers guarantee resource cleanup on both normal and abnormal termination.
- Dead fibers (where `alive?` returns false) raise `FiberError` if you try to resume or raise on them.

## Code Examples

**A timeout wrapper that raises an exception with a cause chain inside a fiber after a deadline — demonstrates Fiber#raise with cause:**

```ruby
# Timeout pattern using Fiber#raise with cause chain (Ruby 4.0)
def with_fiber_timeout(fiber, seconds)
  timer = Thread.new do
    sleep seconds
    if fiber.alive?
      timeout = Timeout::Error.new("Operation exceeded #{seconds}s")
      fiber.raise(timeout, cause: RuntimeError.new("Deadline policy"))
    end
  end

  result = fiber.resume
  timer.kill
  result
end

worker = Fiber.new do
  begin
    Fiber.yield  # Simulate long-running work
  rescue Timeout::Error => e
    puts "#{e.message} (caused by: #{e.cause.message})"
  end
end

worker.resume
with_fiber_timeout(worker, 5)
```


## Resources

- [Fiber Class](https://docs.ruby-lang.org/en/4.0/Fiber.html) — Official Ruby Fiber documentation covering raise, alive?, and error handling

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*