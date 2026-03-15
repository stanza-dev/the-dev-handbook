---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-synchronization-patterns"
---

# Synchronization Patterns

## Introduction
Beyond Mutex and ConditionVariable, the `concurrent-ruby` gem provides higher-level synchronization primitives that solve common coordination problems. ReadWriteLock, Semaphore, CountDownLatch, and CyclicBarrier each address a specific pattern that appears repeatedly in concurrent systems.

## Key Concepts
- **ReadWriteLock**: Allows multiple concurrent readers OR a single exclusive writer. Readers don't block each other, but a writer blocks everyone.
- **Semaphore**: Limits concurrent access to a resource to at most N threads. Think of it as a Mutex that allows N simultaneous holders.
- **CountDownLatch**: Blocks one or more threads until a counter reaches zero. Each event decrements the counter.
- **CyclicBarrier**: Synchronizes N threads at a rendezvous point. All N must arrive before any can proceed. Reusable for multiple rounds.

## Real World Context
A database connection pool uses a Semaphore to limit concurrent connections to the pool size. A deployment script uses CountDownLatch to wait for all microservices to report healthy before switching traffic. A parallel simulation uses CyclicBarrier to synchronize time steps — all workers must finish step N before any starts step N+1.

## Deep Dive

### ReadWriteLock

When reads vastly outnumber writes, a ReadWriteLock dramatically improves throughput over a Mutex:

```ruby
require 'concurrent'

lock = Concurrent::ReadWriteLock.new
cache = {}

# Multiple readers can run simultaneously
readers = 10.times.map do
  Thread.new do
    lock.with_read_lock do
      cache[:exchange_rate]  # Concurrent reads — no blocking
    end
  end
end

# Writer gets exclusive access
writer = Thread.new do
  lock.with_write_lock do
    cache[:exchange_rate] = fetch_latest_rate  # Exclusive write
  end
end

[*readers, writer].each(&:join)
```

`with_read_lock` allows multiple threads inside simultaneously. `with_write_lock` waits for all readers to finish, then blocks new readers until the write completes.

### Semaphore

Semaphore controls how many threads can access a resource concurrently:

```ruby
require 'concurrent'

# Database connection pool: max 5 concurrent connections
db_semaphore = Concurrent::Semaphore.new(5)

requests = 20.times.map do |i|
  Thread.new do
    db_semaphore.acquire
    begin
      puts "Request #{i}: connected (#{5 - db_semaphore.available_permits} active)"
      sleep 0.3  # Simulate query
    ensure
      db_semaphore.release  # Always release, even on error
    end
  end
end

requests.each(&:join)
```

`acquire` blocks when all permits are taken. `release` returns a permit. Always release in an `ensure` block to prevent permit leaks.

### CountDownLatch

Wait for a fixed number of events to occur:

```ruby
require 'concurrent'

service_count = 3
latch = Concurrent::CountDownLatch.new(service_count)

# Each service reports readiness independently
%w[auth payments inventory].each do |service|
  Thread.new do
    sleep rand(0.5..2.0)  # Simulate startup time
    puts "#{service} is ready"
    latch.count_down  # Decrement the counter
  end
end

puts "Waiting for all services..."
latch.wait  # Blocks until count reaches 0
puts "All services ready — starting traffic!"
```

Unlike a barrier, CountDownLatch does not require the counted threads to wait at the latch. They simply decrement and continue.

### CyclicBarrier

Synchronize threads at a rendezvous point, then release them all:

```ruby
require 'concurrent'

worker_count = 3
barrier = Concurrent::CyclicBarrier.new(worker_count)

workers = worker_count.times.map do |id|
  Thread.new do
    3.times do |phase|
      sleep rand(0.1..0.3)  # Simulate varying work times
      puts "Worker #{id}: phase #{phase} complete"
      barrier.wait  # Wait for all workers to reach this point
      puts "Worker #{id}: starting phase #{phase + 1}"
    end
  end
end

workers.each(&:join)
```

The barrier resets automatically after all threads arrive, making it reusable for multiple synchronization rounds (hence "cyclic").

## Common Pitfalls
1. **Forgetting `ensure` with Semaphore** — If a thread raises an exception without releasing its permit, the permit is permanently lost. Always use `ensure` or the block form if available.
2. **Using CyclicBarrier with dynamic thread counts** — The barrier expects exactly N threads. If one thread dies before arriving, the others wait forever. Use a timeout: `barrier.wait(5)` (seconds).

## Best Practices
1. **Choose the right primitive for the job** — Don't use a Semaphore(1) when you need a Mutex, or a CyclicBarrier when you need a CountDownLatch. Each has distinct semantics.
2. **Combine primitives carefully** — Mixing locks can cause deadlocks. Document the lock ordering and keep the scope of each lock as small as possible.
3. **Set timeouts on blocking operations** — `latch.wait(timeout)` and `barrier.wait(timeout)` prevent indefinite hangs in production.

## Summary
- ReadWriteLock: multiple concurrent readers OR one exclusive writer.
- Semaphore: limit concurrent access to N permits.
- CountDownLatch: wait for N events, then proceed (one-shot).
- CyclicBarrier: synchronize N threads at a rendezvous point (reusable).
- Always release Semaphore permits in `ensure` blocks.
- Use timeouts to prevent indefinite waits.

## Code Examples

**Combining CyclicBarrier for phase synchronization with CountDownLatch for completion detection — test suites set up together, then run independently**

```ruby
require 'concurrent'

# Parallel test runner with barrier-synchronized phases
barrier = Concurrent::CyclicBarrier.new(3)
latch = Concurrent::CountDownLatch.new(3)

test_suites = %w[unit integration e2e]

test_suites.each do |suite|
  Thread.new do
    # Phase 1: Setup
    puts "#{suite}: setting up test environment"
    sleep rand(0.1..0.5)
    barrier.wait  # All suites must finish setup before running

    # Phase 2: Run tests
    puts "#{suite}: running tests"
    sleep rand(0.2..0.8)
    latch.count_down  # Signal completion
  end
end

latch.wait
puts "All test suites completed!"
```


## Resources

- [concurrent-ruby Synchronization](https://ruby-concurrency.github.io/concurrent-ruby/master/Concurrent/Semaphore.html) — API reference for Semaphore, ReadWriteLock, CountDownLatch, and CyclicBarrier

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*