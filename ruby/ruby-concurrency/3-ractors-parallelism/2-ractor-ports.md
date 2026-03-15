---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-ractor-ports"
---

# Ractor::Port Communication

## Introduction
Ractor::Port is the primary messaging primitive in Ruby 4.0's Ractor system. Ports replace the removed `Ractor.yield` and `Ractor#take` methods, providing explicit, directional communication channels between Ractors. Understanding ports is essential for building any multi-Ractor system beyond simple compute-and-return patterns.

## Key Concepts
- **Ractor::Port**: A unidirectional message channel. Any Ractor can send to a port; any Ractor holding a reference can receive from it.
- **Default Port**: Every Ractor is born with one port, accessible via `ractor.default_port` or internally via `Ractor.receive`.
- **`port.send(obj)`**: Enqueues an object on the port. The object is deep-copied unless it is shareable or moved.
- **`port.receive`**: Blocks until a message arrives, then returns it.
- **`port.close`**: Closes the port so no more messages can be sent. Receivers get `Ractor::ClosedError`.

## Real World Context
In production pipeline architectures — such as an ETL system that extracts records, transforms them, and loads them into a database — each stage runs in its own Ractor. Ports connect the stages, providing backpressure-free message passing without shared memory. This pattern scales linearly with CPU cores.

## Deep Dive

### Creating and Using Ports

You create a port with `Ractor::Port.new` and pass it into Ractors as an argument:

```ruby
result_port = Ractor::Port.new

worker = Ractor.new(result_port) do |port|
  # Perform work inside the Ractor
  total = (1..100).sum
  # Send the result back through the port
  port.send(total)
end

# Receive the result on the main Ractor
answer = result_port.receive  # => 5050
worker.join
```

The port acts as a mailbox: `worker` sends a value into it, and the main Ractor pulls it out with `receive`. This decouples the sender from the receiver.

### Default Port

Every Ractor has a default port. Sending to a Ractor with `r.send(msg)` is shorthand for `r.default_port.send(msg)`. Inside the Ractor, `Ractor.receive` reads from the default port:

```ruby
worker = Ractor.new do
  # Ractor.receive reads from this Ractor's default port
  name = Ractor.receive
  "Hello, #{name}!"
end

# r.send is shorthand for r.default_port.send
worker.send("Alice")
worker.value  # => "Hello, Alice!"
```

Both forms are equivalent, but explicit ports are preferred when a Ractor needs multiple independent channels.

### Multiple Ports Pattern

Use separate ports for input and output to keep concerns cleanly separated:

```ruby
input_port = Ractor::Port.new
output_port = Ractor::Port.new

processor = Ractor.new(input_port, output_port) do |inp, out|
  while (data = inp.receive)
    out.send(data.transform_keys(&:upcase))
  end
rescue Ractor::ClosedError
  # Input port was closed, stop processing
end

input_port.send({ name: "Alice", role: "admin" })
result = output_port.receive  # => { NAME: "Alice", ROLE: "admin" }

input_port.close  # Signal the worker to stop
processor.join
```

Closing the input port causes `inp.receive` to raise `Ractor::ClosedError`, which the Ractor catches to exit its loop gracefully.

### The `<<` Operator

`Ractor#send` has a shorthand `<<` operator that works identically:

```ruby
worker = Ractor.new { Ractor.receive * 2 }
worker << 21       # Same as worker.send(21)
worker.value       # => 42
```

## Common Pitfalls
1. **Forgetting to close ports** — Long-running Ractors that loop on `port.receive` will hang forever unless the port is closed. Always close input ports when you are done sending.
2. **Sending non-shareable ports** — A `Ractor::Port` is itself a shareable object, so it can be passed to multiple Ractors. But forgetting to pass the port as an argument to `Ractor.new` and trying to capture it from the closure will raise `Ractor::IsolationError`.

## Best Practices
1. **One port per concern** — Use a dedicated port for commands, another for results, and another for errors rather than multiplexing everything on the default port.
2. **Rescue `ClosedError` in loops** — Treat `Ractor::ClosedError` as the normal shutdown signal for worker Ractors that loop on `receive`.

## Summary
- `Ractor::Port.new` creates a message channel between Ractors.
- `port.send(obj)` enqueues; `port.receive` dequeues (blocking).
- Every Ractor has a `default_port`; `r.send(msg)` is shorthand for `r.default_port.send(msg)`.
- Close ports with `port.close` to signal shutdown to looping receivers.
- The `<<` operator is an alias for `send`.

## Code Examples

**Fan-out pattern with a shared task port and result port — three Ractors compete to receive work and report results**

```ruby
# Fan-out pattern: one producer, multiple consumers via ports
task_port = Ractor::Port.new
result_port = Ractor::Port.new

# Spawn 3 worker Ractors sharing the same task port
3.times do |id|
  Ractor.new(task_port, result_port, id) do |tasks, results, worker_id|
    while (url = tasks.receive)
      # Simulate processing a URL
      results.send({ worker: worker_id, url: url, status: 200 })
    end
  rescue Ractor::ClosedError
    # Producer closed the task port, exit gracefully
  end
end

# Enqueue tasks
urls = %w[/users /orders /products /inventory /reports]
urls.each { |url| task_port.send(url) }
task_port.close

# Collect results
urls.size.times { puts result_port.receive.inspect }
```


## Resources

- [Ractor::Port (Ruby 4.0)](https://docs.ruby-lang.org/en/4.0/Ractor/Port.html) — Official documentation for Ractor::Port including send, receive, and close

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*