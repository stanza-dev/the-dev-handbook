---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-thread-lifecycle"
---

# Thread Lifecycle & Exceptions

## Introduction
Threads in Ruby move through distinct states — running, sleeping, aborting, and dead. Understanding these states and how exceptions behave inside threads is crucial for writing reliable concurrent programs. This lesson covers thread states, return values, exception handling, and thread-local storage.

## Key Concepts
- **Thread Status**: A string indicating the thread's current state — `"run"`, `"sleep"`, `"aborting"`, `false` (terminated normally), or `nil` (terminated with exception).
- **Thread#value**: Blocks until the thread completes and returns its final expression, re-raising any unhandled exception.
- **Thread-local Variable**: Data scoped to a specific thread, invisible to other threads.

## Real World Context
In production web servers like Puma, threads handle individual HTTP requests. If a thread raises an exception and it goes unnoticed, the request silently fails with no log entry. Understanding thread exception behavior prevents silent data loss and debugging nightmares.

## Deep Dive

### Thread States

You can inspect a thread's current state at any time using `status` and `alive?`. The following example creates a sleeping thread and checks its state.

```ruby
worker = Thread.new { sleep 10 }

puts worker.status   # => "sleep"
puts worker.alive?   # => true
puts worker.stop?    # => true (sleeping counts as stopped)
```

A thread in the `"sleep"` state is alive but not actively running. It will resume when the sleep period ends or when another thread wakes it.

### Retrieving Return Values

`Thread#value` is the preferred way to get a thread's result. It blocks until the thread finishes, then returns whatever the thread block evaluated to.

```ruby
price_thread = Thread.new do
  # Simulate fetching a price from an API
  sleep 0.5
  99.99
end

price = price_thread.value  # Blocks, then returns 99.99
puts "Price: $#{price}"
```

Using `value` is cleaner than calling `join` followed by accessing a shared variable, because the return value is scoped to the thread.

### Exception Handling

By default, Ruby threads swallow exceptions silently. The exception only surfaces when you call `join` or `value` on the thread.

```ruby
faulty_thread = Thread.new do
  raise "Connection timed out"
end

sleep 0.1
puts "Main thread continues"  # This runs — no crash!

# To see the exception, you must join or call value:
faulty_thread.value  # Raises: RuntimeError: Connection timed out
```

If you want all thread exceptions to crash the program immediately (useful during development), enable `abort_on_exception`.

```ruby
Thread.abort_on_exception = true

Thread.new { raise "This crashes everything" }
sleep 0.1  # Main thread crashes here
```

The global setting affects all threads. For per-thread control, set it on the individual thread instance.

### Thread-Local Variables

Each thread has its own storage. Values set on one thread are invisible to others.

```ruby
Thread.current[:request_id] = "req-abc-123"

background = Thread.new do
  puts Thread.current[:request_id]  # => nil (different thread)
  Thread.current[:request_id] = "req-xyz-789"
  puts Thread.current[:request_id]  # => "req-xyz-789"
end

background.join
puts Thread.current[:request_id]  # => "req-abc-123" (unchanged)
```

Thread-local variables are commonly used to store request context in web frameworks.

## Common Pitfalls
1. **Ignoring thread exceptions** — If you never call `join` or `value`, exceptions vanish silently. Always retrieve thread results or enable `abort_on_exception` during development.
2. **Assuming thread-local variables are inherited** — Child threads do not inherit the parent's thread-local variables. Use `thread_variable_set` / `thread_variable_get` for fiber-local storage that fibers within the same thread share.

## Best Practices
1. **Always call `value` instead of `join` when you need the result** — It combines waiting and result retrieval in one call, and it re-raises exceptions so they are never silently lost.
2. **Use `abort_on_exception` in development** — Silent thread failures make debugging extremely difficult. Turn it on in dev/test environments to catch problems early.

## Summary
- Threads cycle through `"run"`, `"sleep"`, `"aborting"`, `false`, and `nil` states.
- `Thread#value` blocks and returns the thread's result, re-raising any exception.
- Exceptions inside threads are silently swallowed unless you explicitly retrieve them.
- Thread-local variables provide per-thread isolated storage with `Thread.current[:key]`.

## Code Examples

**A thread that fetches data and handles errors internally, returning either parsed data or an error hash**

```ruby
# Safely handle thread results and exceptions
worker = Thread.new do
  response = Net::HTTP.get(URI("https://api.example.com/users"))
  JSON.parse(response)
rescue => error
  { error: error.message }
end

result = worker.value
if result.is_a?(Hash) && result[:error]
  puts "Request failed: #{result[:error]}"
else
  puts "Got #{result.length} users"
end
```


## Resources

- [Thread Class](https://docs.ruby-lang.org/en/4.0/Thread.html) — Official Ruby Thread class reference covering states, exceptions, and thread-local variables

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*