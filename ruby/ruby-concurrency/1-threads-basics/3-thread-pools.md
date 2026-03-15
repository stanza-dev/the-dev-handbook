---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-thread-pools"
---

# Thread Pools

## Introduction
Creating a new thread for every task is expensive — each thread allocates roughly 1MB of stack memory and requires OS scheduling overhead. Thread pools solve this by maintaining a fixed set of reusable worker threads that pull jobs from a shared queue. This lesson covers building a pool from scratch and using the `concurrent-ruby` gem.

## Key Concepts
- **Thread Pool**: A collection of pre-created threads that process tasks from a work queue, avoiding the cost of repeated thread creation and destruction.
- **Work Queue**: A thread-safe data structure (Ruby's `Queue` class) where producers enqueue tasks and pool workers dequeue and execute them.
- **Future**: A placeholder for a result that will be available later, allowing you to fire off a computation and retrieve the value when needed.

## Real World Context
Puma, Ruby's most popular web server, uses a thread pool to handle HTTP requests. Sidekiq uses thread pools for background job processing. Without pooling, a traffic spike that spawns thousands of threads would exhaust system memory and crash the process.

## Deep Dive

### Building a Thread Pool with Queue

Ruby's built-in `Queue` class is thread-safe, making it perfect for distributing work. The following pool creates a fixed number of workers that pull jobs off the queue until they receive a `nil` sentinel (poison pill).

```ruby
class TaskPool
  def initialize(size)
    @size = size
    @queue = Queue.new
    @workers = size.times.map do
      Thread.new do
        while (task = @queue.pop)  # Blocks until a task arrives
          task.call
        end
      end
    end
  end

  def schedule(&block)
    @queue << block
  end

  def shutdown
    @size.times { @queue << nil }  # One poison pill per worker
    @workers.each(&:join)
  end
end

pool = TaskPool.new(4)
20.times { |i| pool.schedule { puts "Processing order #{i}" } }
pool.shutdown
```

Each worker thread loops on `@queue.pop`, which blocks when the queue is empty. When `shutdown` pushes `nil` for each worker, the `while` condition becomes false and the thread exits.

### Using concurrent-ruby

The `concurrent-ruby` gem provides production-grade pool implementations with error handling, timeouts, and backpressure.

```ruby
require 'concurrent'

pool = Concurrent::FixedThreadPool.new(5)

20.times do |i|
  pool.post { puts "Processing order #{i}" }
end

pool.shutdown
pool.wait_for_termination
```

The `post` method submits a block to the pool. `shutdown` stops accepting new tasks, and `wait_for_termination` blocks until all queued tasks complete.

### Futures for Async Results

When you need the result of an asynchronous computation, use `Concurrent::Future`. It runs the block in a thread pool and lets you retrieve the result later.

```ruby
require 'concurrent'

user_future = Concurrent::Future.execute do
  # Simulate API call
  sleep 1
  { name: "Alice", email: "alice@example.com" }
end

puts user_future.state       # => :pending
user_data = user_future.value  # Blocks until complete
puts user_data[:name]        # => "Alice"
puts user_future.fulfilled?  # => true
```

The `value` method blocks until the future resolves. If the block raises an exception, `value` returns `nil` and you can inspect the error with `reason`.

## Common Pitfalls
1. **Creating unbounded pools** — A pool that grows without limit under load is no better than spawning raw threads. Always set a maximum size that matches your system's capacity.
2. **Forgetting to shut down** — If you don't call `shutdown` and `wait_for_termination`, the program may exit before pool tasks complete, losing their results.

## Best Practices
1. **Size pools to your workload** — For I/O-bound tasks, pool sizes of 5-20 work well. For CPU-bound tasks under the GVL, more threads won't help — consider Ractors instead.
2. **Use `concurrent-ruby` in production** — Hand-rolled pools lack proper error handling, shutdown semantics, and backpressure. The gem handles all of this correctly.

## Summary
- Thread pools reuse a fixed set of threads, avoiding the overhead of creating and destroying threads per task.
- Ruby's `Queue` class provides the thread-safe backbone for a simple pool implementation.
- The `concurrent-ruby` gem offers `FixedThreadPool` and `Future` for production-grade async patterns.

## Code Examples

**Concurrent API fetching using Futures — each request runs in a pool thread while the main thread waits for all results**

```ruby
require 'concurrent'

# Fetch multiple URLs concurrently using futures
urls = [
  "https://api.example.com/users",
  "https://api.example.com/orders",
  "https://api.example.com/products"
]

futures = urls.map do |url|
  Concurrent::Future.execute do
    Net::HTTP.get(URI(url))  # Each runs in the thread pool
  end
end

# Collect all results (blocks until each is done)
responses = futures.map(&:value)
puts "Fetched #{responses.length} API responses"
```


## Resources

- [concurrent-ruby gem](https://github.com/ruby-concurrency/concurrent-ruby) — Production-grade concurrency tools for Ruby including thread pools, futures, and atomic references

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*