---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-condition-variables"
---

# Condition Variables

## Introduction
When threads need to wait for a specific condition before proceeding — such as a consumer waiting for a producer to generate data — a Mutex alone is insufficient. You need a way to signal between threads. ConditionVariable provides this signaling mechanism, allowing threads to sleep efficiently until they are explicitly woken.

## Key Concepts
- **ConditionVariable**: A synchronization primitive that lets threads wait for a signal from another thread. It always works in conjunction with a Mutex.
- **`cv.wait(mutex)`**: Atomically releases the mutex, sleeps until signaled, then re-acquires the mutex before returning.
- **`cv.signal`**: Wakes exactly one waiting thread.
- **`cv.broadcast`**: Wakes all waiting threads.
- **Spurious Wakeup**: A thread may occasionally wake from `wait` even when no `signal` was sent. Always re-check the condition in a loop.

## Real World Context
Consider a background job processor where worker threads wait for new jobs to appear in a shared list. Without a ConditionVariable, workers would need to busy-loop checking the list, wasting CPU. With a ConditionVariable, workers sleep until the producer signals that a new job is available, achieving near-zero CPU usage while idle.

## Deep Dive

### Basic Producer-Consumer Pattern

The canonical use of ConditionVariable is the producer-consumer pattern, where one thread generates data and another processes it:

```ruby
mutex = Mutex.new
cv = ConditionVariable.new
queue = []

consumer = Thread.new do
  mutex.synchronize do
    while queue.empty?
      cv.wait(mutex)  # Release mutex, sleep, re-acquire on wake
    end
    item = queue.shift
    puts "Consumed: #{item}"
  end
end

producer = Thread.new do
  sleep 0.5  # Simulate work
  mutex.synchronize do
    queue << "job_payload"
    cv.signal  # Wake the consumer
  end
end

[producer, consumer].each(&:join)
```

The consumer holds the mutex, checks if the queue is empty, and if so calls `cv.wait(mutex)`. This atomically releases the mutex (so the producer can acquire it) and puts the consumer to sleep. When the producer signals, the consumer wakes up, re-acquires the mutex, and proceeds.

### Why `while` and Not `if`?

Spurious wakeups mean a thread can wake from `cv.wait` without any call to `signal` or `broadcast`. The condition might also change between the signal and the re-acquisition of the mutex. Always use a loop:

```ruby
# WRONG — may process an empty queue after spurious wakeup
if queue.empty?
  cv.wait(mutex)
end
item = queue.shift  # Could be nil!

# CORRECT — re-checks condition after every wakeup
while queue.empty?
  cv.wait(mutex)
end
item = queue.shift  # Guaranteed non-nil
```

### Signal vs Broadcast

`signal` wakes exactly one waiting thread. `broadcast` wakes all of them:

```ruby
mutex = Mutex.new
cv = ConditionVariable.new
ready = false

# Multiple waiting threads
waiters = 3.times.map do |i|
  Thread.new do
    mutex.synchronize do
      until ready
        cv.wait(mutex)
      end
      puts "Thread #{i} proceeding"
    end
  end
end

sleep 0.1
mutex.synchronize do
  ready = true
  cv.broadcast  # Wake ALL waiters
end

waiters.each(&:join)
```

Use `signal` when only one waiter should proceed (e.g., one consumer per item). Use `broadcast` when all waiters should re-evaluate their condition (e.g., a shutdown signal).

### Internal Mechanics of `cv.wait(mutex)`

Understanding the three-step atomic sequence is crucial:
1. Release the mutex (so other threads can modify the shared state)
2. Sleep until signaled
3. Re-acquire the mutex before returning

This atomicity prevents a race where the signal arrives between releasing the mutex and sleeping.

## Common Pitfalls
1. **Using `if` instead of `while`** — Spurious wakeups or multiple waiters competing for the same item can cause your code to process stale or missing data.
2. **Forgetting to hold the mutex when calling `signal`** — While not strictly required by Ruby, signaling without the mutex can cause missed signals if the consumer checks the condition and calls `wait` between the state change and the signal.

## Best Practices
1. **Always pair ConditionVariable with a predicate** — The condition (e.g., `queue.empty?`) is the real synchronization mechanism; the ConditionVariable just avoids busy-waiting.
2. **Prefer Ruby's `Queue` class for simple cases** — `Queue` encapsulates the mutex + ConditionVariable pattern internally. Use raw ConditionVariable only when you need custom coordination logic.

## Summary
- ConditionVariable lets threads sleep until signaled, avoiding busy-waiting.
- Always check the condition in a `while` loop, not an `if`, to handle spurious wakeups.
- `signal` wakes one waiter; `broadcast` wakes all.
- `cv.wait(mutex)` atomically releases the mutex, sleeps, then re-acquires.
- For simple producer-consumer, prefer Ruby's built-in `Queue` class.

## Code Examples

**Bounded buffer using two ConditionVariables — not_full prevents the producer from overfilling, not_empty prevents the consumer from reading an empty buffer**

```ruby
# Bounded buffer with ConditionVariable for backpressure
mutex = Mutex.new
not_full = ConditionVariable.new
not_empty = ConditionVariable.new
buffer = []
max_size = 5

producer = Thread.new do
  20.times do |i|
    mutex.synchronize do
      while buffer.size >= max_size
        not_full.wait(mutex)  # Wait until buffer has room
      end
      buffer << "item_#{i}"
      not_empty.signal  # Notify consumer that buffer is non-empty
    end
  end
end

consumer = Thread.new do
  20.times do
    mutex.synchronize do
      while buffer.empty?
        not_empty.wait(mutex)  # Wait until buffer has items
      end
      item = buffer.shift
      not_full.signal  # Notify producer that buffer has room
      puts "Processed: #{item}"
    end
  end
end

[producer, consumer].each(&:join)
```


## Resources

- [ConditionVariable Class](https://docs.ruby-lang.org/en/4.0/Thread/ConditionVariable.html) — Official Ruby documentation for ConditionVariable including wait, signal, and broadcast

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*