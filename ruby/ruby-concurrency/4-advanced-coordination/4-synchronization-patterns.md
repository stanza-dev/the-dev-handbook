---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-synchronization-patterns"
---

# Common Concurrency Patterns

## 1. Read-Write Lock

Multiple readers OR one writer:

```ruby
require 'concurrent'

lock = Concurrent::ReadWriteLock.new
data = {}

# Multiple readers can access simultaneously
Thread.new do
  lock.with_read_lock do
    puts data[:key]
  end
end

# Writer has exclusive access
Thread.new do
  lock.with_write_lock do
    data[:key] = "value"
  end
end
```

## 2. Semaphore

Limit concurrent access to a resource:

```ruby
require 'concurrent'

# Allow max 3 concurrent connections
sem = Concurrent::Semaphore.new(3)

10.times.map do
  Thread.new do
    sem.acquire
    begin
      use_limited_resource
    ensure
      sem.release
    end
  end
end.each(&:join)
```

## 3. CountDownLatch

Wait for N events:

```ruby
latch = Concurrent::CountDownLatch.new(3)

3.times do |i|
  Thread.new do
    sleep rand
    puts "Worker #{i} done"
    latch.count_down
  end
end

latch.wait  # Blocks until count reaches 0
puts "All workers finished!"
```

## 4. Barrier

Synchronize N threads at a point:

```ruby
barrier = Concurrent::CyclicBarrier.new(3)

3.times do |i|
  Thread.new do
    puts "#{i}: Phase 1 done"
    barrier.wait  # All threads wait here
    puts "#{i}: Phase 2 starting"
  end
end
```

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*