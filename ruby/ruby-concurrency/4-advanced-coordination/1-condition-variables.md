---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-condition-variables"
---

# Coordination with ConditionVariable

Sometimes threads need to wait for a signal from another thread.

## Basic Pattern

```ruby
mutex = Mutex.new
cv = ConditionVariable.new
ready = false

# Consumer thread
consumer = Thread.new do
  mutex.synchronize do
    until ready
      cv.wait(mutex)  # Releases mutex while waiting
    end
    puts "Data is ready!"
  end
end

# Producer thread
producer = Thread.new do
  sleep 1  # Simulate work
  mutex.synchronize do
    ready = true
    cv.signal  # Wake ONE waiting thread
  end
end

[producer, consumer].each(&:join)
```

## signal vs broadcast

```ruby
cv.signal     # Wake ONE waiting thread
cv.broadcast  # Wake ALL waiting threads
```

## Why Pass Mutex to wait?

The mutex must be released while waiting, otherwise no one can signal!

```ruby
# cv.wait(mutex) does:
# 1. Release mutex
# 2. Sleep until signaled
# 3. Re-acquire mutex before returning
```

## Spurious Wakeups

Always use a loop, not `if`:

```ruby
# WRONG - might wake without condition being true
if !ready
  cv.wait(mutex)
end

# CORRECT - re-check condition after wake
until ready
  cv.wait(mutex)
end
```

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*