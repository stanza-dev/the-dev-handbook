---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-atomic-operations"
---

# Atomic References

Some operations need to be atomic (indivisible) without full mutex overhead.

## concurrent-ruby Atomics

```ruby
require 'concurrent'

# Atomic reference
ref = Concurrent::AtomicReference.new(0)

threads = 10.times.map do
  Thread.new do
    1000.times { ref.update { |v| v + 1 } }
  end
end

threads.each(&:join)
ref.value  # => 10000 (always!)
```

## AtomicFixnum (Optimized for Integers)

```ruby
counter = Concurrent::AtomicFixnum.new(0)

counter.increment       # => 1
counter.increment(5)    # => 6
counter.decrement       # => 5
counter.compare_and_set(5, 10)  # => true, sets to 10 if currently 5
```

## Compare and Swap (CAS)

```ruby
ref = Concurrent::AtomicReference.new("initial")

# Only update if current value matches expected
ref.compare_and_set("initial", "updated")  # => true
ref.compare_and_set("initial", "nope")     # => false (value changed)
```

## When to Use Atomics vs Mutex

| Use Atomics when | Use Mutex when |
|------------------|----------------|
| Single value update | Multiple values |
| Simple increment/CAS | Complex operations |
| High contention | Longer critical sections |
| Performance critical | Simpler to reason about |

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*