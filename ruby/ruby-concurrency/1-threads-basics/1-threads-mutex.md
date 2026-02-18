---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-threads-mutex"
---

# The Global VM Lock (GVL)

MRI (CRuby) has a Global VM Lock (formerly GIL). Only one thread can execute Ruby code at a time.

**However**, threads are still useful for:
- I/O-bound operations (network, disk)
- Waiting operations (sleep, select)
- C extensions that release the GVL

## Creating Threads

```ruby
thread = Thread.new do
  puts "Hello from thread!"
end

thread.join  # Wait for thread to finish
```

## Race Conditions

Without synchronization, shared data can be corrupted:

```ruby
counter = 0

threads = 10.times.map do
  Thread.new do
    1000.times { counter += 1 }  # NOT thread-safe!
  end
end

threads.each(&:join)
puts counter  # Often less than 10000!
```

## Mutex for Safety

Use `Mutex` to synchronize access:

```ruby
mutex = Mutex.new
counter = 0

threads = 10.times.map do
  Thread.new do
    1000.times do
      mutex.synchronize do
        counter += 1  # Now thread-safe
      end
    end
  end
end

threads.each(&:join)
puts counter  # Always 10000
```

## Resources

- [Thread Class](https://docs.ruby-lang.org/en/4.0/Thread.html) — Ruby Thread documentation

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*