---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-thread-pools"
---

# Why Thread Pools?

Creating threads is expensive. Thread pools reuse a fixed number of threads.

## Simple Thread Pool

```ruby
class ThreadPool
  def initialize(size)
    @size = size
    @queue = Queue.new
    @threads = size.times.map do
      Thread.new do
        while (job = @queue.pop)
          job.call
        end
      end
    end
  end

  def schedule(&block)
    @queue << block
  end

  def shutdown
    @size.times { @queue << nil }  # Poison pills
    @threads.each(&:join)
  end
end

pool = ThreadPool.new(4)
10.times { |i| pool.schedule { puts i } }
pool.shutdown
```

## Using concurrent-ruby Gem

```ruby
require 'concurrent'

pool = Concurrent::FixedThreadPool.new(5)

10.times do |i|
  pool.post { puts "Job #{i}" }
end

pool.shutdown
pool.wait_for_termination
```

## Futures

```ruby
require 'concurrent'

future = Concurrent::Future.execute do
  sleep 1
  42
end

future.state      # => :pending
future.value      # Blocks until complete, returns 42
future.fulfilled? # => true
```

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*