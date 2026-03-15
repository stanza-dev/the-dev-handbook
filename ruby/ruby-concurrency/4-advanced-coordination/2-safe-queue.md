---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-thread-safe-queue"
---

# Thread-Safe Queues

## Introduction
Ruby's standard library provides `Queue` and `SizedQueue` — thread-safe data structures built specifically for producer-consumer patterns. They handle all the Mutex and ConditionVariable mechanics internally, letting you focus on business logic rather than synchronization details.

## Key Concepts
- **`Queue`**: An unbounded, thread-safe FIFO queue. `pop` blocks when empty; `push` never blocks.
- **`SizedQueue`**: A bounded version of Queue. `push` blocks when the queue reaches its maximum size, providing automatic backpressure.
- **Blocking pop**: `queue.pop` suspends the calling thread until an item is available.
- **Non-blocking pop**: `queue.pop(true)` raises `ThreadError` immediately if the queue is empty.
- **`queue.close`**: Prevents future pushes. After closing, `pop` returns `nil` once the queue drains.

## Real World Context
Web servers use thread-safe queues to decouple request acceptance from processing. A listener thread pushes incoming requests onto a Queue, and a pool of worker threads pops and processes them. SizedQueue prevents memory exhaustion during traffic spikes by making the listener wait when the queue is full, applying natural backpressure to the client.

## Deep Dive

### Basic Queue Usage

The simplest producer-consumer pattern needs just a Queue:

```ruby
queue = Queue.new

producer = Thread.new do
  ["order_101", "order_102", "order_103"].each do |order|
    queue.push(order)
    puts "Enqueued: #{order}"
  end
  queue.close  # Signal that no more items will be added
end

consumer = Thread.new do
  while (order = queue.pop)  # Returns nil after close + drain
    puts "Processing: #{order}"
    sleep 0.1  # Simulate work
  end
  puts "Queue closed, consumer exiting"
end

[producer, consumer].each(&:join)
```

When the queue is closed and empty, `pop` returns `nil`, which terminates the `while` loop cleanly. This is the idiomatic shutdown pattern — no poison pills needed.

### Non-Blocking Operations

Sometimes you want to check for items without blocking:

```ruby
queue = Queue.new
queue.push("task_a")

queue.pop             # => "task_a" (would block if empty)

begin
  queue.pop(true)     # => raises ThreadError (non_block = true)
rescue ThreadError
  puts "Queue is empty, moving on"
end

queue.empty?          # => true
queue.size            # => 0 (also aliased as .length)
```

The `true` argument to `pop` enables non-blocking mode. This is useful in event loops where you want to check for work without stalling.

### SizedQueue for Backpressure

SizedQueue limits the number of items in the queue. When full, `push` blocks until a consumer pops an item:

```ruby
queue = SizedQueue.new(3)  # Maximum 3 items

producer = Thread.new do
  10.times do |i|
    queue.push("batch_#{i}")  # Blocks when queue has 3 items!
    puts "Produced: batch_#{i}"
  end
  queue.close
end

consumer = Thread.new do
  while (item = queue.pop)
    sleep 0.2  # Slow consumer
    puts "Consumed: #{item}"
  end
end

[producer, consumer].each(&:join)
```

The producer runs ahead until the queue fills to 3, then blocks until the slow consumer frees a slot. This prevents unbounded memory growth.

### Multiple Consumers

Queue naturally supports multiple consumers — each `pop` atomically dequeues exactly one item:

```ruby
queue = Queue.new
100.times { |i| queue.push("job_#{i}") }
queue.close

workers = 4.times.map do |id|
  Thread.new do
    count = 0
    while (job = queue.pop)
      count += 1
      # Process job...
    end
    puts "Worker #{id} processed #{count} jobs"
  end
end

workers.each(&:join)
```

No two workers ever receive the same item. The work distributes naturally based on processing speed.

### ClosedQueueError

Pushing to a closed queue raises `ClosedQueueError`:

```ruby
queue = Queue.new
queue.close

begin
  queue.push("late_item")
rescue ClosedQueueError
  puts "Cannot push to a closed queue"
end
```

## Common Pitfalls
1. **Using poison pills instead of `close`** — The old pattern of pushing `nil` or a special sentinel to signal shutdown is fragile. Use `queue.close` instead, which is atomic and works correctly with multiple consumers.
2. **Checking `empty?` before `pop`** — This creates a race condition: another thread can pop the last item between your `empty?` check and your `pop` call. Use blocking `pop` directly.

## Best Practices
1. **Use `SizedQueue` for production systems** — Unbounded queues can cause out-of-memory errors under load. Always set a reasonable bound.
2. **Close the queue from the producer side** — The producer knows when it is done; closing the queue propagates this knowledge to all consumers cleanly.
3. **Prefer `while (item = queue.pop)` for the consumer loop** — This idiom handles both the normal items and the `nil` return after close in a single expression.

## Summary
- `Queue` is an unbounded, thread-safe FIFO; `SizedQueue` adds a capacity limit.
- `pop` blocks when empty; `pop(true)` raises `ThreadError` instead.
- `push` blocks on `SizedQueue` when full, providing backpressure.
- `close` prevents further pushes; `pop` returns `nil` after the queue drains.
- Multiple consumers can safely share one Queue — each item is delivered exactly once.

## Code Examples

**Three-stage log processing pipeline using SizedQueue for backpressure at each stage — reader, parser workers, and aggregator**

```ruby
# Log processing pipeline with SizedQueue backpressure
raw_queue = SizedQueue.new(100)
parsed_queue = SizedQueue.new(50)

# Stage 1: Reader
reader = Thread.new do
  File.foreach("/var/log/app.log") do |line|
    raw_queue.push(line.chomp)
  end
  raw_queue.close
end

# Stage 2: Parser (2 workers)
parsers = 2.times.map do
  Thread.new do
    while (line = raw_queue.pop)
      timestamp, level, message = line.split(" ", 3)
      parsed_queue.push({ timestamp: timestamp, level: level, message: message })
    end
  end
end

# Wait for parsers, then close the parsed queue
Thread.new { parsers.each(&:join); parsed_queue.close }

# Stage 3: Aggregator
while (entry = parsed_queue.pop)
  # Store in database, forward to monitoring, etc.
end
```


## Resources

- [Queue Class](https://docs.ruby-lang.org/en/4.0/Thread/Queue.html) — Official Ruby Queue documentation including pop, push, close, and SizedQueue

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*