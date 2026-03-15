---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-production-patterns"
---

# Production Concurrency Patterns

## Introduction
Knowing how Threads, Fibers, and Ractors work is only half the battle. Production systems need patterns for controlling throughput, handling failures gracefully, and shutting down cleanly. This lesson covers four battle-tested patterns that turn raw concurrency primitives into production-ready infrastructure.

## Key Concepts
- **Backpressure**: A mechanism that slows down producers when consumers cannot keep up, preventing unbounded memory growth.
- **Circuit breaker**: A pattern that stops calling a failing service after repeated errors, giving it time to recover before retrying.
- **Graceful shutdown**: Stopping workers so that in-flight work completes and no data is lost, typically triggered by OS signals.
- **Connection pooling**: Sharing a fixed set of expensive resources (database connections, HTTP clients) across multiple workers.

## Real World Context
Without backpressure, a fast producer can fill a queue with millions of items, exhausting memory and crashing the process. Without graceful shutdown, deploying a new version kills in-flight jobs, corrupting data. Without a circuit breaker, one failing microservice cascades failures across your entire system. These patterns are not optional in production — they are required.

## Deep Dive

### Pattern 1: Backpressure with SizedQueue

Ruby's `Thread::SizedQueue` blocks the producer when the queue is full, automatically applying backpressure:

```ruby
# SizedQueue with capacity 10 — producer blocks when full
queue = Thread::SizedQueue.new(10)

producer = Thread.new do
  1000.times do |i|
    queue << "job_#{i}"  # Blocks when queue has 10 items
    puts "Produced job_#{i}"
  end
  queue << :done
end

consumer = Thread.new do
  while (job = queue.pop) != :done
    sleep 0.01  # Simulate slow processing
    puts "Processed #{job}"
  end
end

[producer, consumer].each(&:join)
```

The `SizedQueue` acts as a buffer with a maximum size. When the queue reaches capacity, the producer's `<<` call blocks until the consumer removes an item. This keeps memory usage bounded regardless of how fast the producer generates work.

You can also check the queue state without blocking:

```ruby
queue = Thread::SizedQueue.new(5)
queue.size      # Current number of items
queue.max       # Maximum capacity (5)
queue.empty?    # true if no items
queue.num_waiting  # Number of threads waiting to push or pop
```

### Pattern 2: Circuit Breaker

A circuit breaker tracks failures and stops calling a service when errors exceed a threshold:

```ruby
class CircuitBreaker
  STATES = %i[closed open half_open].freeze

  def initialize(threshold:, timeout:)
    @threshold = threshold
    @timeout = timeout
    @failure_count = 0
    @state = :closed
    @last_failure_time = nil
    @mutex = Mutex.new
  end

  def call(&block)
    @mutex.synchronize do
      case @state
      when :open
        if Time.now - @last_failure_time >= @timeout
          @state = :half_open
        else
          raise CircuitOpenError, "Circuit is open — service unavailable"
        end
      end
    end

    begin
      result = block.call
      @mutex.synchronize { reset }
      result
    rescue StandardError => e
      @mutex.synchronize { record_failure }
      raise
    end
  end

  private

  def record_failure
    @failure_count += 1
    @last_failure_time = Time.now
    @state = :open if @failure_count >= @threshold
  end

  def reset
    @failure_count = 0
    @state = :closed
  end
end
```

The circuit starts **closed** (requests flow normally). After `threshold` consecutive failures, it **opens** (all requests are immediately rejected). After `timeout` seconds, it moves to **half-open** (one request is allowed through to test if the service recovered). A successful request resets the circuit to closed.

### Pattern 3: Graceful Shutdown with Signal Handling

Production processes receive OS signals when it is time to stop. A graceful shutdown finishes in-flight work before exiting:

```ruby
class GracefulWorker
  def initialize(pool_size:)
    @queue = Thread::Queue.new
    @running = true
    @workers = pool_size.times.map { |i| spawn_worker(i) }
    setup_signal_handlers
  end

  def schedule(job)
    @queue << job if @running
  end

  def wait
    @workers.each(&:join)
  end

  private

  def setup_signal_handlers
    %w[INT TERM].each do |signal|
      Signal.trap(signal) do
        puts "\nReceived #{signal}, shutting down gracefully..."
        @running = false
        @workers.size.times { @queue << :shutdown }
      end
    end
  end

  def spawn_worker(id)
    Thread.new do
      while (job = @queue.pop) != :shutdown
        begin
          job.call
        rescue StandardError => e
          $stderr.puts "Worker #{id} error: #{e.message}"
        end
      end
      puts "Worker #{id} shut down cleanly"
    end
  end
end
```

When a SIGINT or SIGTERM arrives, the signal handler sets `@running = false` to stop accepting new jobs and pushes `:shutdown` sentinel values into the queue. Each worker finishes its current job, pops the `:shutdown` sentinel, and exits its loop cleanly.

### Pattern 4: Connection Pooling

Expensive resources like database connections should be pooled and shared across workers:

```ruby
class ConnectionPool
  def initialize(size:, &block)
    @pool = Thread::SizedQueue.new(size)
    size.times { @pool << block.call }
    @mutex = Mutex.new
  end

  def with_connection
    conn = @pool.pop  # Blocks if all connections are in use
    begin
      yield conn
    ensure
      @pool << conn  # Always return connection to pool
    end
  end

  def size
    @pool.size
  end
end

# Usage:
pool = ConnectionPool.new(size: 5) { PG::Connection.new(dbname: "myapp") }

threads = 20.times.map do
  Thread.new do
    pool.with_connection do |conn|
      conn.exec("SELECT * FROM orders WHERE status = 'pending'")
    end
  end
end

threads.each(&:join)
```

The pool uses a `SizedQueue` to store available connections. `with_connection` pops a connection (blocking if none available), yields it to the caller, and returns it in the `ensure` block — guaranteeing the connection is never leaked, even if an exception occurs.

## Common Pitfalls
1. **Unbounded queues in production** — Using `Thread::Queue` instead of `Thread::SizedQueue` lets memory grow without limit. Always set a maximum queue size in production.
2. **Signal handlers that do too much** — Signal handlers run in a special context with restrictions. Keep them minimal — set a flag or push a sentinel, then let the main thread do the real work.
3. **Leaking pooled connections** — If you pop a connection but do not return it in an `ensure` block, an exception will permanently remove that connection from the pool.

## Best Practices
1. **Size queues based on memory budget** — Calculate the maximum item size multiplied by the queue capacity. A 1000-item queue of 1 KB objects uses about 1 MB, which is predictable and safe.
2. **Log circuit breaker state changes** — When a circuit opens or closes, log it with the failure count and timeout. This makes debugging service outages much easier.
3. **Test shutdown with SIGTERM** — In your integration tests, send `Process.kill('TERM', Process.pid)` and verify that all in-flight work completes and no data is lost.

## Summary
- SizedQueue provides automatic backpressure by blocking producers when the queue is full.
- The circuit breaker pattern prevents cascading failures by cutting off requests to a failing service.
- Graceful shutdown uses signal handlers and sentinel values to finish in-flight work before exiting.
- Connection pooling with SizedQueue and ensure blocks shares expensive resources safely across workers.

## Code Examples

**SizedQueue backpressure — the producer blocks when the queue reaches capacity, keeping memory bounded while all 20 items are eventually processed**

```ruby
# Backpressure in action: SizedQueue limits memory usage
queue = Thread::SizedQueue.new(5)

producer = Thread.new do
  20.times do |i|
    queue << "item_#{i}"  # Blocks when 5 items are queued
  end
  queue << :stop
end

consumer = Thread.new do
  processed = 0
  while (item = queue.pop) != :stop
    processed += 1
  end
  puts "Processed #{processed} items"  # => Processed 20 items
end

[producer, consumer].each(&:join)
# Memory never exceeded 5 items despite producing 20
```

**Signal-based graceful shutdown — the worker finishes its current job before exiting when SIGTERM is received**

```ruby
# Graceful shutdown with signal trapping
running = true
queue = Thread::Queue.new

Signal.trap("TERM") do
  running = false
  queue << :shutdown
end

worker = Thread.new do
  while (job = queue.pop) != :shutdown
    process(job)  # Finishes current job before stopping
  end
  puts "Worker exited cleanly"
end

# In production, SIGTERM is sent by the process manager
# worker.join ensures the main process waits for cleanup
```


## Resources

- [Thread::SizedQueue Documentation](https://docs.ruby-lang.org/en/4.0/Thread/SizedQueue.html) — Official reference for Ruby's bounded thread-safe queue that provides automatic backpressure
- [Signal Module Documentation](https://docs.ruby-lang.org/en/4.0/Signal.html) — Ruby's Signal module for trapping OS signals like SIGTERM and SIGINT in production processes

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*