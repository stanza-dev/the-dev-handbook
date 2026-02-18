---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-thread-lifecycle"
---

# Thread States

Threads go through several states:

```ruby
thread = Thread.new { sleep 10 }

thread.status  # => "sleep" (or "run", "aborting", false, nil)
thread.alive?  # => true
thread.stop?   # => true (sleeping)
```

## Join and Value

Always `join` threads you care about:

```ruby
thread = Thread.new { 42 }
thread.join   # Wait for completion
thread.value  # => 42 (the return value)

# Or combine:
result = Thread.new { expensive_calculation }.value
```

## Exception Handling

By default, threads **swallow exceptions**:

```ruby
thread = Thread.new { raise "Error!" }
thread.join  # Nothing happens!

# To make exceptions visible:
Thread.abort_on_exception = true
# OR
thread = Thread.new { raise "Error!" }
thread.join  # Now raises!
# OR check value:
thread.value  # Re-raises the exception
```

## Thread Local Variables

```ruby
Thread.current[:user_id] = 123

Thread.new do
  Thread.current[:user_id]  # => nil (different thread!)
end

# Fiber-local (inherited by fibers):
Thread.current.thread_variable_set(:request_id, "abc")
Thread.current.thread_variable_get(:request_id)
```

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*