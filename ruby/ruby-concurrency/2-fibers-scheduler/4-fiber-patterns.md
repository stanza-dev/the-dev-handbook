---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-fiber-patterns"
---

# Fiber Patterns

## Introduction
Fibers enable several powerful concurrency patterns that are impossible or awkward with threads alone. This lesson covers four production-ready patterns: producer/consumer coroutines, multi-stage processing pipelines, state machines, and cooperative task scheduling. Each pattern exploits fibers' cooperative nature for clean, predictable code.

## Key Concepts
- **Coroutine**: Two or more fibers that cooperatively pass data back and forth, each yielding to let the other run.
- **Pipeline**: A chain of fibers where each stage transforms data and passes it to the next, processing elements lazily one at a time.
- **State Machine**: A fiber that cycles through a fixed set of states, yielding the current state on each resume.

## Real World Context
ETL (Extract, Transform, Load) systems use pipeline patterns to process millions of records with constant memory usage — each stage processes one record, yields it downstream, and requests the next. Chat protocols and game loops use state machines to model conversation states or turn sequences. The `Enumerator::Lazy` class in Ruby's standard library is essentially a fiber pipeline.

## Deep Dive

### Pattern 1: Producer/Consumer Coroutine

A producer fiber generates items and yields them one by one. A consumer loop resumes the producer to pull each item.

```ruby
def order_producer(orders)
  Fiber.new do
    orders.each do |order|
      puts "Producing order: #{order[:id]}"
      Fiber.yield order
    end
    nil  # Signal completion
  end
end

def process_orders(producer_fiber)
  while (order = producer_fiber.resume)
    puts "Shipping order #{order[:id]} to #{order[:customer]}"
  end
  puts "All orders processed"
end

orders = [
  { id: 101, customer: "Alice", total: 59.99 },
  { id: 102, customer: "Bob", total: 129.00 },
  { id: 103, customer: "Carol", total: 34.50 }
]

process_orders(order_producer(orders))
# Output:
# Producing order: 101
# Shipping order 101 to Alice
# Producing order: 102
# Shipping order 102 to Bob
# Producing order: 103
# Shipping order 103 to Carol
# All orders processed
```

Notice the interleaved output: the producer generates one item at a time, and the consumer processes it before the next item is produced. This keeps memory usage constant regardless of collection size.

### Pattern 2: Processing Pipeline

Chain multiple fiber stages together. Each stage takes a fiber as input, processes each value, and yields the transformed result.

```ruby
def double_stage(source_fiber)
  Fiber.new do
    while (value = source_fiber.resume)
      Fiber.yield value * 2
    end
  end
end

def add_tax_stage(source_fiber, tax_rate)
  Fiber.new do
    while (price = source_fiber.resume)
      Fiber.yield (price * (1 + tax_rate)).round(2)
    end
  end
end

# Source: produces base prices
source = Fiber.new do
  [10.00, 25.00, 50.00].each { |price| Fiber.yield price }
  nil
end

# Build pipeline: source → double → add 8% tax
pipeline = add_tax_stage(double_stage(source), 0.08)

while (final_price = pipeline.resume)
  puts "Final price: $#{final_price}"
end
# Output:
# Final price: $21.6
# Final price: $54.0
# Final price: $108.0
```

Each stage processes one value at a time, so even with millions of records, memory usage stays flat. This is the same principle behind Unix pipes.

### Pattern 3: State Machine

A fiber naturally models a state machine. It yields the current state and transitions to the next one on each resume.

```ruby
def order_state_machine
  Fiber.new do
    loop do
      Fiber.yield :pending
      Fiber.yield :payment_processing
      Fiber.yield :confirmed
      Fiber.yield :shipped
      Fiber.yield :delivered
    end
  end
end

order_status = order_state_machine
5.times do
  state = order_status.resume
  puts "Order state: #{state}"
end
# Output:
# Order state: pending
# Order state: payment_processing
# Order state: confirmed
# Order state: shipped
# Order state: delivered
```

The fiber remembers where it is in the cycle. Each resume advances to the next state. No instance variables or case statements needed.

### Pattern 4: Cooperative Task Scheduler

You can build a simple round-robin scheduler that rotates through fibers, giving each a turn.

```ruby
def cooperative_scheduler(tasks)
  fibers = tasks.map { |task| Fiber.new(&task) }

  until fibers.empty?
    fibers.reject! do |fiber|
      fiber.resume
      !fiber.alive?
    end
  end
end

task_a = proc do
  3.times do |i|
    puts "Task A: step #{i}"
    Fiber.yield
  end
end

task_b = proc do
  2.times do |i|
    puts "Task B: step #{i}"
    Fiber.yield
  end
end

cooperative_scheduler([task_a, task_b])
# Output:
# Task A: step 0
# Task B: step 0
# Task A: step 1
# Task B: step 1
# Task A: step 2
```

The scheduler resumes each fiber in turn. When a fiber finishes (no longer alive), it is removed from the rotation. This is the core idea behind Ruby's Fiber::Scheduler.

## Common Pitfalls
1. **Infinite generators without exit conditions** — A producer fiber that never returns `nil` causes the consumer to loop forever. Always define a clear completion signal.
2. **Deep pipeline chains** — Each pipeline stage adds a fiber to the call stack. Beyond 10-15 stages, consider using `Enumerator::Lazy` which is optimized for this use case.

## Best Practices
1. **Use `nil` as the completion signal** — Return `nil` from the fiber's last expression to signal "no more data." This convention is used throughout Ruby's Enumerator implementation.
2. **Favor `Enumerator::Lazy` for data pipelines** — It composes better with Ruby's Enumerable methods and handles edge cases (empty input, errors) more robustly than raw fiber pipelines.

## Summary
- Producer/consumer coroutines process data one item at a time with constant memory.
- Pipeline patterns chain fiber stages like Unix pipes for lazy multi-step transformations.
- State machines map naturally to fibers — each resume advances to the next state.
- A cooperative scheduler gives each fiber a turn, forming the conceptual basis of Fiber::Scheduler.

## Code Examples

**An ETL (Extract, Transform, Load) pipeline using chained fibers — each record flows through stages one at a time with constant memory**

```ruby
# ETL pipeline with fibers: extract → transform → load
def extract(records)
  Fiber.new do
    records.each { |record| Fiber.yield record }
    nil
  end
end

def transform(source)
  Fiber.new do
    while (record = source.resume)
      Fiber.yield record.merge(processed_at: Time.now.utc)
    end
  end
end

raw_data = [{ name: "Alice", age: 30 }, { name: "Bob", age: 25 }]
pipeline = transform(extract(raw_data))
while (result = pipeline.resume)
  puts "Loaded: #{result[:name]} (processed at #{result[:processed_at]})"
end
```


## Resources

- [Fiber Class](https://docs.ruby-lang.org/en/4.0/Fiber.html) — Official Ruby Fiber documentation covering cooperative concurrency patterns

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*