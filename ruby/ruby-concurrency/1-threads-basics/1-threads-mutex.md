---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-threads-mutex"
---

# Threads & Mutex

## Introduction
Ruby's MRI interpreter uses a Global VM Lock (GVL, formerly GIL) that allows only one thread to execute Ruby code at a time. Despite this limitation, threads remain essential for I/O-bound workloads where waiting dominates execution. This lesson covers thread creation, race conditions, and how Mutex protects shared state.

## Key Concepts
- **Global VM Lock (GVL)**: A lock in MRI Ruby that prevents multiple threads from executing Ruby code simultaneously, though it is released during I/O operations.
- **Race Condition**: A bug where the program's correctness depends on the unpredictable timing of thread execution.
- **Mutex**: A mutual exclusion lock that ensures only one thread can execute a critical section at a time.

## Real World Context
Every web server handling concurrent requests (Puma, Unicorn) relies on threads. When two requests update the same database record or modify a shared cache, race conditions cause data corruption. Mutex and other synchronization primitives are the standard defense.

## Deep Dive

### Creating Threads

The following code spawns a new thread and waits for it to finish with `join`. Without `join`, the main thread might exit before the spawned thread completes.

```ruby
download_thread = Thread.new do
  puts "Downloading file..."
  # Simulates network I/O — GVL is released during sleep
  sleep 2
  puts "Download complete"
end

download_thread.join  # Wait for thread to finish
puts "File ready"
# Output:
# Downloading file...
# Download complete
# File ready
```

The `Thread.new` block runs concurrently. During `sleep` (which simulates I/O), the GVL is released and other threads can run.

### Race Conditions Without Synchronization

When multiple threads modify the same variable without synchronization, results become unpredictable. The increment operation `counter += 1` is not atomic — it reads, increments, then writes, and another thread can interleave between those steps.

```ruby
account_balance = 0

deposit_threads = 10.times.map do
  Thread.new do
    1000.times { account_balance += 1 }  # NOT thread-safe!
  end
end

deposit_threads.each(&:join)
puts account_balance  # Often less than 10000!
```

The final value is unpredictable because threads read stale values of `account_balance` while another thread is mid-update.

### Fixing with Mutex

A `Mutex` wraps the critical section so only one thread at a time can execute it. Every other thread blocks on `synchronize` until the lock is released.

```ruby
balance_lock = Mutex.new
account_balance = 0

deposit_threads = 10.times.map do
  Thread.new do
    1000.times do
      balance_lock.synchronize do
        account_balance += 1  # Now thread-safe
      end
    end
  end
end

deposit_threads.each(&:join)
puts account_balance  # Always 10000
```

The `synchronize` method acquires the lock, runs the block, then releases the lock — even if an exception occurs inside the block.

## Common Pitfalls
1. **Forgetting to join threads** — If the main thread exits before spawned threads finish, their work is lost. Always call `join` or `value` on threads you care about.
2. **Locking too broadly** — Wrapping large sections of code in `synchronize` eliminates parallelism. Lock only the minimal critical section that accesses shared state.

## Best Practices
1. **Prefer high-level abstractions** — Use `Queue`, `concurrent-ruby` gems, or thread pools rather than raw Mutex when possible. They encapsulate synchronization correctly.
2. **Keep shared mutable state minimal** — The less data threads share, the fewer synchronization bugs you will encounter. Design for isolation first, synchronization second.

## Summary
- The GVL prevents parallel Ruby execution but is released during I/O, making threads useful for network and disk operations.
- Race conditions occur when threads read-modify-write shared data without synchronization.
- `Mutex#synchronize` is the fundamental tool for protecting critical sections in Ruby.

## Code Examples

**A Mutex guarantees that only one thread increments the counter at a time, preventing race conditions**

```ruby
# Thread-safe counter using Mutex
balance_lock = Mutex.new
account_balance = 0

workers = 5.times.map do
  Thread.new do
    100.times do
      balance_lock.synchronize { account_balance += 1 }
    end
  end
end

workers.each(&:join)
puts account_balance  # => 500 (always correct)
```


## Resources

- [Thread Class](https://docs.ruby-lang.org/en/4.0/Thread.html) — Ruby Thread documentation

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*