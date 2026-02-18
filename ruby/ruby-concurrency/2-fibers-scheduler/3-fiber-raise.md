---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-fiber-raise"
---

# Fiber#raise (Ruby 4.0 Enhanced)

You can raise exceptions inside a fiber from outside.

## Basic raise

```ruby
fiber = Fiber.new do
  begin
    loop { Fiber.yield }
  rescue => e
    puts "Caught: #{e.message}"
  end
end

fiber.resume
fiber.raise(RuntimeError, "Stop!")  # "Caught: Stop!"
```

## Ruby 4.0: raise with cause

```ruby
original_error = StandardError.new("Original")

fiber = Fiber.new do
  begin
    Fiber.yield
  rescue => e
    puts e.message    # "Wrapped"
    puts e.cause.message  # "Original"
  end
end

fiber.resume
fiber.raise(RuntimeError, "Wrapped", cause: original_error)
```

## Fiber Cleanup

```ruby
fiber = Fiber.new do
  begin
    resource = acquire_resource
    loop { Fiber.yield process(resource) }
  ensure
    release_resource(resource)  # Always runs!
  end
end

fiber.resume
fiber.resume
fiber.raise(StopIteration)  # Triggers ensure block
```

## Dead Fiber Detection

```ruby
fiber = Fiber.new { "done" }
fiber.resume  # => "done"

fiber.alive?  # => false
fiber.resume  # FiberError: dead fiber called
```

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*