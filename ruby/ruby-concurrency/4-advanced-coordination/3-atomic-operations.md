---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-atomic-operations"
---

# Atomic Operations

## Introduction
Mutexes are powerful but carry overhead: acquiring and releasing locks, potential contention, and risk of deadlocks. For simple operations like incrementing a counter or swapping a reference, atomic operations provide a lighter-weight, lock-free alternative. The `concurrent-ruby` gem offers production-grade atomic primitives used widely in Rails and other Ruby frameworks.

## Key Concepts
- **Atomic Operation**: An operation that completes in a single, indivisible step from the perspective of other threads. No other thread can observe a half-completed state.
- **Compare-and-Swap (CAS)**: Atomically updates a value only if it currently equals an expected value. If another thread changed it first, CAS fails and you retry.
- **`Concurrent::AtomicReference`**: A thread-safe container for any object, supporting atomic reads, writes, and CAS.
- **`Concurrent::AtomicFixnum`**: Optimized for integer operations: increment, decrement, and CAS.

## Real World Context
Rate limiters, hit counters, and feature flags all need thread-safe updates that happen millions of times per second. Using a Mutex for each increment creates a bottleneck. Atomic operations let multiple threads update counters concurrently with minimal overhead, which is why Rails uses `Concurrent::AtomicFixnum` internally for request counting.

## Deep Dive

### AtomicReference

`AtomicReference` wraps any object with thread-safe access:

```ruby
require 'concurrent'

ref = Concurrent::AtomicReference.new(0)

threads = 10.times.map do
  Thread.new do
    1000.times do
      ref.update { |current| current + 1 }  # Atomic read-modify-write
    end
  end
end

threads.each(&:join)
puts ref.value  # => 10000 (always correct, no Mutex needed)
```

The `update` method reads the current value, applies the block, and writes the result — all atomically. If another thread modified the value between read and write, `update` automatically retries.

### AtomicFixnum

For integer-only operations, `AtomicFixnum` is faster than `AtomicReference`:

```ruby
counter = Concurrent::AtomicFixnum.new(0)

counter.increment         # => 1
counter.increment(5)      # => 6 (add 5)
counter.decrement         # => 5
counter.value             # => 5

# CAS: only update if current value matches expected
counter.compare_and_set(5, 100)   # => true (was 5, now 100)
counter.compare_and_set(5, 200)   # => false (is 100, not 5)
```

`compare_and_set` is the building block for lock-free algorithms. It returns `true` if the swap succeeded, `false` if the value had already changed.

### Compare-and-Swap Pattern

CAS is the foundation of lock-free programming. Here is a retry loop implementing an atomic maximum:

```ruby
max_value = Concurrent::AtomicReference.new(0)

def atomic_max(ref, candidate)
  loop do
    current = ref.value
    return current if candidate <= current  # Already >= candidate
    return candidate if ref.compare_and_set(current, candidate)  # CAS succeeded
    # CAS failed — another thread changed the value. Retry.
  end
end

threads = 100.times.map do |i|
  Thread.new { atomic_max(max_value, rand(1000)) }
end

threads.each(&:join)
puts max_value.value  # Correct maximum from 100 random values
```

The loop retries until the CAS succeeds, guaranteeing correctness without locks.

### AtomicBoolean

For flags and toggles:

```ruby
shutdown = Concurrent::AtomicBoolean.new(false)

worker = Thread.new do
  until shutdown.true?
    process_next_item
  end
  puts "Worker shutting down"
end

sleep 5
shutdown.make_true  # Atomically set to true
worker.join
```

### When to Use Atomics vs Mutex

The choice depends on the complexity of the critical section:

| Use Atomics when | Use Mutex when |
|------------------|----------------|
| Updating a single value | Updating multiple related values |
| Simple increment/CAS | Complex multi-step operations |
| High contention (many threads competing) | Longer critical sections |
| Performance-critical hot paths | Simpler to reason about |

## Common Pitfalls
1. **Using atomics for multi-variable updates** — If you need to update two counters together (e.g., total and count for an average), atomics on each variable separately won't be atomic as a pair. Use a Mutex to group them.
2. **Spinning on CAS without backoff** — Under extreme contention, CAS retry loops can waste CPU. Consider using `update` (which handles retries) instead of manual CAS loops.

## Best Practices
1. **Prefer `update` over manual CAS loops** — `AtomicReference#update` handles the retry logic for you and is less error-prone.
2. **Use `AtomicFixnum` for counters** — It is optimized for integer math and faster than wrapping an integer in `AtomicReference`.
3. **Default to Mutex and optimize to atomics** — Start with a Mutex for clarity. Only switch to atomics when profiling shows lock contention is a bottleneck.

## Summary
- Atomic operations update values in one indivisible step, avoiding locks.
- `Concurrent::AtomicFixnum` handles thread-safe integer math with `increment`, `decrement`, and `compare_and_set`.
- `Concurrent::AtomicReference` wraps any object with atomic `update` and CAS.
- `update` retries automatically on contention; prefer it over manual CAS loops.
- Use atomics for single-value hot paths; use Mutex for multi-value critical sections.

## Code Examples

**Rate limiter combining AtomicFixnum for the request counter and AtomicReference for the window timestamp — lock-free and safe under high concurrency**

```ruby
require 'concurrent'

# Thread-safe rate limiter using AtomicFixnum
class RateLimiter
  def initialize(max_per_second)
    @max = max_per_second
    @count = Concurrent::AtomicFixnum.new(0)
    @window_start = Concurrent::AtomicReference.new(Time.now)
  end

  def allow?
    now = Time.now
    start = @window_start.value

    if now - start >= 1.0
      # New window — reset counter atomically
      if @window_start.compare_and_set(start, now)
        @count.value = 1
        return true
      end
    end

    @count.increment <= @max
  end
end

limiter = RateLimiter.new(100)
puts limiter.allow?  # => true (first request in window)
```


## Resources

- [concurrent-ruby Atomics](https://ruby-concurrency.github.io/concurrent-ruby/master/Concurrent/AtomicFixnum.html) — API reference for AtomicFixnum including increment, decrement, and compare_and_set

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*