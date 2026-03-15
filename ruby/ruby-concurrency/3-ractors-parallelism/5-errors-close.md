---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-ractor-errors-close"
---

# Ractor Errors and Lifecycle

## Introduction
Building reliable concurrent systems requires understanding how errors propagate across Ractor boundaries, how Ractors terminate, and what guarantees the lifecycle methods provide. Ruby 4.0 introduces a clear error hierarchy and strict rules about closing Ractors that differ significantly from thread error handling.

## Key Concepts
- **`Ractor::IsolationError`**: Raised when code violates Ractor isolation rules, such as accessing an outer-scope variable from inside a Ractor block.
- **`Ractor::MovedError`**: Raised when accessing an object that was moved to another Ractor via `send(obj, move: true)`.
- **`Ractor::ClosedError`**: Raised when sending to or receiving from a closed port.
- **`Ractor::UnsafeError`**: Raised when attempting to share an object that is not safe to share between Ractors.
- **`Ractor::RemoteError`**: Wraps an exception raised inside a Ractor; raised on the caller's side when calling `join` or `value` on a Ractor that terminated with an unhandled exception.
- **`Ractor#close`**: Closes the current Ractor. In Ruby 4.0, `close` only works when called on `Ractor.current`.

## Real World Context
In a pipeline of Ractors — say, a log processing system where one Ractor reads, another parses, and a third aggregates — any stage can fail. Without proper error handling, a crash in the parser Ractor silently drops all logs. Understanding `RemoteError` lets the aggregator detect the failure and either retry or alert operators.

## Deep Dive

### Error Propagation with RemoteError

When a Ractor's block raises an unhandled exception, it terminates. The exception is captured and re-raised as `Ractor::RemoteError` when the caller invokes `join` or `value`:

```ruby
faulty = Ractor.new do
  raise ArgumentError, "invalid input: negative age"
end

begin
  faulty.join
rescue Ractor::RemoteError => e
  puts e.message         # => "thrown by remote Ractor"
  puts e.cause.class     # => ArgumentError
  puts e.cause.message   # => "invalid input: negative age"
end
```

The original exception is available via `e.cause`, allowing the caller to inspect the actual error. This two-layer design keeps the error's origin clear.

### IsolationError

Attempting to access outer-scope variables from a Ractor block raises `IsolationError`:

```ruby
greeting = "hello"

begin
  Ractor.new { puts greeting }  # Tries to capture 'greeting'
rescue Ractor::IsolationError => e
  puts e.message  # => can not access non-shareable objects from Ractor
end

# Fix: pass as argument
Ractor.new(greeting) { |g| puts g }  # Deep-copies greeting
```

This error is raised at Ractor creation time, not at runtime inside the block, making it easy to catch during development.

### Closing Ractors and Ports

In Ruby 4.0, `Ractor#close` is restricted — it only works on the current Ractor:

```ruby
worker = Ractor.new do
  Ractor.receive  # Wait for a message
  Ractor.current.close  # Close self — OK
end

worker.send(:start)
worker.join

# Attempting to close a different Ractor raises an error
# worker.close  # => Error! Can only close Ractor.current
```

To stop a remote Ractor, close the port it reads from, which causes `Ractor::ClosedError` inside the Ractor:

```ruby
task_port = Ractor::Port.new

worker = Ractor.new(task_port) do |port|
  loop { port.receive }
rescue Ractor::ClosedError
  # Port was closed externally, shut down gracefully
end

task_port.close  # Triggers ClosedError inside the worker
worker.join
```

### UnsafeError

Some objects are inherently unsafe to share or send. Attempting to do so raises `UnsafeError`:

```ruby
io = File.open("/tmp/test.txt", "w")

begin
  Ractor.new(io) { |f| f.write("hello") }
rescue Ractor::UnsafeError => e
  puts e.message  # => can not pass IO objects between Ractors
end
```

IO objects, Threads, and other resource handles cannot cross Ractor boundaries because they are tied to OS resources that cannot be safely shared.

## Common Pitfalls
1. **Ignoring RemoteError** — If you never call `join` or `value` on a Ractor, its exception is silently lost. Always join Ractors you care about.
2. **Trying to close a remote Ractor directly** — In Ruby 4.0, `close` only works on `Ractor.current`. Use port closing or a poison-pill message to signal shutdown instead.

## Best Practices
1. **Always rescue RemoteError around `join`/`value`** — Treat it like rescuing exceptions around `Thread#value`. Inspect `e.cause` for the real error.
2. **Use port closing for clean shutdown** — Close the input port to trigger `ClosedError` in the worker's receive loop. This is the idiomatic Ruby 4.0 shutdown pattern.
3. **Validate data before sending** — Check `Ractor.shareable?(obj)` before sharing, and avoid sending IO objects, Procs (unless shareable), or Thread references.

## Summary
- `Ractor::RemoteError` wraps exceptions from terminated Ractors; access the original via `e.cause`.
- `IsolationError` fires at Ractor creation when the block captures non-shareable outer variables.
- `MovedError` fires when accessing an object sent with `move: true`.
- `ClosedError` fires when a closed port is used for send or receive.
- `UnsafeError` fires when trying to send un-sendable objects like IO handles.
- `Ractor#close` only works on `Ractor.current` — use port closing to stop remote Ractors.

## Code Examples

**Error-resilient Ractor that catches exceptions and sends error details through the result port instead of crashing silently**

```ruby
# Graceful error handling in a Ractor pipeline
result_port = Ractor::Port.new

worker = Ractor.new(result_port) do |port|
  data = Ractor.receive
  processed = data.map { |record| record[:amount] * 1.08 }  # Add tax
  port.send(processed)
rescue => e
  port.send({ error: e.class.name, message: e.message })
end

worker.send([{ amount: 100 }, { amount: 200 }])
result = result_port.receive

if result.is_a?(Hash) && result[:error]
  puts "Worker failed: #{result[:error]} — #{result[:message]}"
else
  puts "Processed: #{result.inspect}"  # => [108.0, 216.0]
end
```


## Resources

- [Ractor Error Handling](https://docs.ruby-lang.org/en/4.0/Ractor.html) — Ractor docs covering RemoteError, IsolationError, and lifecycle management

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*