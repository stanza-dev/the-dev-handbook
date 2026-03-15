---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-choosing-tool"
---

# Choosing the Right Tool

## Introduction
Ruby offers three major concurrency primitives — Threads, Fibers, and Ractors — each with distinct strengths and trade-offs. Choosing the wrong one leads to wasted effort, poor performance, or subtle bugs. This lesson gives you a decision framework so you can pick the right tool for every concurrency problem you face.

## Key Concepts
- **I/O-bound work**: Tasks that spend most of their time waiting for external resources (network, disk, database). The CPU is idle during the wait.
- **CPU-bound work**: Tasks that perform heavy computation (image processing, encryption, data crunching). The CPU is fully utilized.
- **Shared-state coordination**: Situations where multiple execution contexts must read and write the same data safely.
- **Concurrency primitive**: A building block provided by the language runtime for managing concurrent execution (Thread, Fiber, Ractor).

## Real World Context
A web application might need all three primitives in different places: Fibers for handling hundreds of concurrent HTTP client requests without blocking, Ractors for parallelizing a CPU-heavy PDF generation pipeline, and Threads with a Mutex for coordinating access to a shared in-memory cache. Picking the right tool for each scenario is the difference between a responsive app and one that crawls under load.

## Deep Dive

The decision starts with one question: **what is the bottleneck?**

### Decision Framework

Here is a comparison table summarizing when to reach for each primitive:

| Criterion | Threads | Fibers / Async | Ractors |
|-----------|---------|----------------|---------|
| Best for | Shared-state coordination, legacy libraries | High-volume I/O (HTTP, DB, file) | CPU-bound parallelism |
| GVL impact | Blocked by GVL for Ruby code | Single-threaded, no GVL concern | Each Ractor has its own GVL |
| Overhead | ~1 MB stack per thread | ~4 KB per fiber | Heavyweight (isolated heap) |
| Data sharing | Shared memory (requires Mutex) | Same thread, no races | No shared mutable state |
| Max practical count | Tens to low hundreds | Thousands to millions | One per CPU core |
| Error model | Exceptions swallowed by default | Exceptions propagate to caller | Exceptions sent via port |

### Path 1: I/O-bound work → Fibers and async

When your code spends most of its time waiting for network responses, database queries, or file reads, Fibers with a scheduler give you the best throughput per thread. You can run thousands of concurrent operations with minimal memory.

The following example shows how the async gem handles concurrent HTTP fetches:

```ruby
require 'async'
require 'async/http/internet'

Async do |task|
  internet = Async::HTTP::Internet.new
  urls = %w[
    https://api.example.com/users
    https://api.example.com/orders
    https://api.example.com/products
  ]

  results = urls.map do |url|
    task.async { internet.get(url) }
  end

  results.each { |r| puts r.wait.status }
ensure
  internet&.close
end
```

All three requests run concurrently on a single thread. No Mutex needed because there is no preemptive scheduling — each fiber yields explicitly when it hits I/O.

### Path 2: CPU-bound work → Ractors

When you need true parallelism for computation, Ractors are the only option in MRI Ruby. Each Ractor runs on its own OS thread with its own GVL, so multiple Ractors execute Ruby code simultaneously.

Here is a parallel prime-checking example using Ractors:

```ruby
ranges = [
  (2..250_000),
  (250_001..500_000),
  (500_001..750_000),
  (750_001..1_000_000)
]

ractors = ranges.map do |range|
  Ractor.new(range) do |r|
    r.select { |n| (2..Math.sqrt(n).to_i).none? { |d| n % d == 0 } }
  end
end

primes = ractors.flat_map(&:value)
puts "Found #{primes.size} primes"
```

Each Ractor processes its range independently. The `value` call blocks until the Ractor finishes, then returns its result. No shared state means no locks.

### Path 3: Shared-state coordination → Threads + Mutex

When multiple workers must read and write the same data structure — a cache, a counter, a rate limiter — Threads with explicit synchronization are the most straightforward choice.

This example shows a thread-safe rate limiter:

```ruby
class RateLimiter
  def initialize(max_per_second)
    @max = max_per_second
    @mutex = Mutex.new
    @count = 0
    @window_start = Time.now
  end

  def allow?
    @mutex.synchronize do
      reset_window_if_needed
      if @count < @max
        @count += 1
        true
      else
        false
      end
    end
  end

  private

  def reset_window_if_needed
    elapsed = Time.now - @window_start
    if elapsed >= 1.0
      @count = 0
      @window_start = Time.now
    end
  end
end
```

The Mutex guarantees that only one thread inspects and modifies the counter at a time, preventing race conditions.

### Hybrid Approaches

Real applications often combine primitives. A common pattern is Ractors for CPU work that feed results into a shared Thread-safe Queue consumed by the main thread:

```ruby
result_queue = Thread::Queue.new

ractor = Ractor.new do
  perform_computation  # Return value available via Ractor#value
end

# Main thread collects Ractor output into shared queue
result_queue << ractor.value
```

This separates the parallel computation from the shared-state coordination cleanly.

## Common Pitfalls
1. **Using Threads for CPU-bound work** — Because of the GVL, multiple threads running pure Ruby computation gain zero speedup. Profile first and switch to Ractors if CPU is the bottleneck.
2. **Using Ractors for I/O** — Ractors have heavy initialization cost and strict isolation rules. For I/O concurrency, Fibers are far lighter and simpler.
3. **Defaulting to the concurrent-ruby gem for everything** — While convenient, adding a dependency when the standard library suffices increases complexity. Evaluate whether Thread, Fiber, or Ractor already covers your use case.

## Best Practices
1. **Profile before choosing** — Use `Benchmark.measure` or a profiler to determine whether your workload is I/O-bound or CPU-bound before selecting a primitive.
2. **Start simple, scale up** — Begin with synchronous code. Add Fibers for I/O concurrency. Only reach for Ractors when you have measured a CPU bottleneck.
3. **Isolate concurrency at boundaries** — Keep concurrent logic at the edges of your system (controllers, job runners) rather than deep inside business logic.

## Summary
- Use Fibers and async for I/O-bound concurrency — they are lightweight and avoid locking.
- Use Ractors for CPU-bound parallelism — they bypass the GVL with true OS-level parallelism.
- Use Threads with Mutex for shared-state coordination where multiple workers touch the same data.
- Profile your workload before choosing a primitive.
- Combine primitives in hybrid architectures when a single tool does not fit.

## Code Examples

**A decision helper illustrating which concurrency primitive to choose based on workload type — I/O-bound, CPU-bound, or shared-state**

```ruby
# Decision helper: choose the right concurrency primitive
def choose_primitive(workload_type)
  case workload_type
  when :io_bound
    # Thousands of concurrent I/O operations with minimal memory
    :fiber_async
  when :cpu_bound
    # True parallelism across CPU cores
    :ractor
  when :shared_state
    # Multiple workers reading/writing shared data
    :thread_with_mutex
  when :hybrid
    # CPU work in Ractors, coordination via Thread::Queue
    :ractor_plus_thread
  end
end

puts choose_primitive(:io_bound)     # => :fiber_async
puts choose_primitive(:cpu_bound)    # => :ractor
puts choose_primitive(:shared_state) # => :thread_with_mutex
```


## Resources

- [Thread Class Documentation](https://docs.ruby-lang.org/en/4.0/Thread.html) — Official Ruby 4.0 Thread class reference covering lifecycle, synchronization, and thread-local variables
- [Ractor Class Documentation](https://docs.ruby-lang.org/en/4.0/Ractor.html) — Official Ruby 4.0 Ractor reference covering ports, isolation, and parallel execution

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*