---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-ractor-select"
---

# Ractor.select, Monitoring, and Storage

## Introduction
When you have multiple Ractors producing results or multiple ports carrying messages, you need a way to wait for whichever is ready first. `Ractor.select` is Ruby 4.0's multiplexing primitive. Combined with monitoring and ractor-local storage, it gives you the tools to build robust, production-grade concurrent systems.

## Key Concepts
- **`Ractor.select(*ractors_or_ports)`**: Blocks until any of the given Ractors terminates or any of the given Ports has a message. Returns `[source, value]`.
- **`Ractor#monitor(port)`**: Registers a port to receive a notification when the monitored Ractor terminates.
- **`Ractor#unmonitor(port)`**: Cancels a previously registered monitor.
- **`Ractor[key]` / `Ractor[key]=`**: Ractor-local storage, similar to thread-local variables but scoped to the current Ractor.
- **`Ractor.store_if_absent(key) { block }`**: Atomically initializes a ractor-local value only if the key is not yet set — thread-safe lazy initialization.

## Real World Context
A task scheduler dispatches jobs to a pool of worker Ractors. It uses `Ractor.select` to detect which worker finishes first and immediately assigns the next job. Monitors let the scheduler detect crashed workers and replace them. Ractor-local storage holds per-worker caches or connection pools that are not shared across workers.

## Deep Dive

### Selecting Across Ractors

Pass Ractor objects to `Ractor.select` to wait for the first one to terminate:

```ruby
workers = 3.times.map do |i|
  Ractor.new(i) do |n|
    sleep(rand(0.1..0.5))  # Simulate varying work durations
    n ** 2
  end
end

# Wait for the first worker to finish
source, result = Ractor.select(*workers)
puts "First result: #{result} from #{source.inspect}"
```

`Ractor.select` returns a two-element array: the Ractor (or Port) that was ready, and the value it produced. For Ractors, the value is the block's return value.

### Selecting Across Ports

You can also select across Port objects to wait for the first message:

```ruby
status_port = Ractor::Port.new
error_port = Ractor::Port.new

Ractor.new(status_port) do |port|
  sleep 0.1
  port.send({ healthy: true, latency_ms: 42 })
end

Ractor.new(error_port) do |port|
  sleep 0.3
  port.send({ error: "timeout", service: "payments" })
end

# Whichever port receives a message first wins
source, message = Ractor.select(status_port, error_port)
if source == error_port
  puts "Error: #{message[:error]}"
else
  puts "Status: latency=#{message[:latency_ms]}ms"
end
```

This is the Ruby 4.0 equivalent of Go's `select` statement — you can multiplex across any combination of Ractors and Ports.

### Collecting All Results

To gather results from all workers, call `select` in a loop:

```ruby
workers = 5.times.map do |i|
  Ractor.new(i) { |n| n * 10 }
end

results = []
remaining = workers.dup

while remaining.any?
  source, value = Ractor.select(*remaining)
  results << value
  remaining.delete(source)
end

puts results.sort  # => [0, 10, 20, 30, 40]
```

Each call to `select` removes the finished Ractor from the remaining list, ensuring we collect exactly one result per worker.

### Monitoring Ractors

`Ractor#monitor(port)` registers a port to receive a notification when the Ractor terminates:

```ruby
monitor_port = Ractor::Port.new

worker = Ractor.new { sleep 0.5; "done" }
worker.monitor(monitor_port)

# Do other work while waiting...
notification = monitor_port.receive
puts "Worker terminated: #{notification.inspect}"

# Cancel monitoring (if Ractor is still alive)
# worker.unmonitor(monitor_port)
```

Monitoring is essential for supervisor patterns where a coordinator Ractor needs to detect and restart failed workers.

### Ractor-Local Storage

Each Ractor has its own key-value store, accessed via `Ractor[key]`:

```ruby
worker = Ractor.new do
  Ractor[:request_count] = 0

  3.times do
    Ractor[:request_count] += 1
  end

  Ractor[:request_count]
end

worker.value  # => 3

# Main Ractor has its own independent storage
Ractor[:request_count]  # => nil (not shared)
```

For thread-safe lazy initialization, use `Ractor.store_if_absent`:

```ruby
worker = Ractor.new do
  # Only initializes once, even if called concurrently from fibers
  cache = Ractor.store_if_absent(:cache) { {} }
  cache[:initialized_at] = Time.now
  cache
end
```

## Common Pitfalls
1. **Selecting on an empty list** — Calling `Ractor.select` with no arguments raises `ArgumentError`. Always verify the list is non-empty before selecting.
2. **Forgetting to remove finished Ractors** — If you select in a loop without removing completed Ractors, you get errors when selecting on a terminated Ractor that has already been consumed.

## Best Practices
1. **Use select for fan-in** — When multiple producers send results to different ports, `Ractor.select` across the ports to process results as they arrive rather than in a fixed order.
2. **Monitor for fault tolerance** — In long-running systems, monitor all worker Ractors so you can restart any that crash unexpectedly.
3. **Use `store_if_absent` for lazy caches** — It provides atomic initialization, preventing double-init races when fibers run inside a Ractor.

## Summary
- `Ractor.select(*ractors_or_ports)` waits for the first ready source and returns `[source, value]`.
- Loop with select and remove finished sources to collect all results.
- `Ractor#monitor(port)` sends a notification to the port when the Ractor terminates.
- `Ractor[key]` provides ractor-local key-value storage; `Ractor.store_if_absent(key) { }` initializes atomically.
- Combining select, monitors, and storage enables supervisor patterns and fault-tolerant architectures.

## Code Examples

**Supervisor pattern using Ractor#monitor — the monitor port receives notifications when workers terminate, enabling automatic restarts**

```ruby
# Supervisor pattern: monitor workers and restart on failure
monitor_port = Ractor::Port.new

def start_worker(id, monitor_port)
  w = Ractor.new(id) do |worker_id|
    Ractor[:worker_id] = worker_id
    loop do
      task = Ractor.receive
      # Process task...
    end
  rescue Ractor::ClosedError
    # Normal shutdown
  end
  w.monitor(monitor_port)
  w
end

workers = 3.times.map { |id| start_worker(id, monitor_port) }

# In a real system, you'd select on monitor_port
# and restart any worker that terminates unexpectedly
```


## Resources

- [Ractor.select (Ruby 4.0)](https://docs.ruby-lang.org/en/4.0/Ractor.html#method-c-select) — Official docs for Ractor.select covering Ractor and Port multiplexing

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*