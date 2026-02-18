---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-thread-safe-queue"
---

# Queue: Built-in Thread Safety

Ruby's `Queue` class is thread-safe by default - perfect for producer/consumer patterns.

## Basic Usage

```ruby
queue = Queue.new

# Producer
producer = Thread.new do
  5.times do |i|
    queue.push("job #{i}")
    sleep 0.1
  end
  queue.push(nil)  # Signal completion
end

# Consumer
consumer = Thread.new do
  while (job = queue.pop)  # Blocks if empty
    puts "Processing: #{job}"
  end
  puts "Done!"
end

[producer, consumer].each(&:join)
```

## Non-blocking Operations

```ruby
queue = Queue.new

queue.push(1)
queue.pop             # => 1 (blocks if empty)
queue.pop(true)       # => raises ThreadError if empty

queue.empty?          # => true
queue.size            # => 0
queue.clear           # Remove all items
```

## SizedQueue: Bounded Buffer

```ruby
# Queue with maximum size
queue = SizedQueue.new(5)  # Max 5 items

producer = Thread.new do
  100.times do |i|
    queue.push(i)  # Blocks when queue is full!
    puts "Produced: #{i}"
  end
end

consumer = Thread.new do
  100.times do
    item = queue.pop
    sleep 0.01  # Slow consumer
    puts "Consumed: #{item}"
  end
end
```

## ClosedQueueError

```ruby
queue = Queue.new
queue.close
queue.push(1)  # ClosedQueueError!
queue.pop      # Returns nil immediately when closed and empty
```

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*