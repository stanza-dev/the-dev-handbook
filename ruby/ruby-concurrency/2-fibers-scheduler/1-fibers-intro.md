---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-fibers-intro"
---

# Fibers

Fibers are primitives for cooperative (non-preemptive) concurrency. Unlike threads, YOU control when fibers switch.

## Basic Fiber Usage

```ruby
fiber = Fiber.new do
  puts "Start"
  Fiber.yield "paused"  # Return control to caller
  puts "Resumed"
  "done"
end

puts fiber.resume  # "Start", returns "paused"
puts fiber.resume  # "Resumed", returns "done"
```

## Two-Way Communication

```ruby
fiber = Fiber.new do |first|
  puts "Received: #{first}"
  second = Fiber.yield "first response"
  puts "Received: #{second}"
  "final"
end

puts fiber.resume("hello")   # Prints: Received: hello
                              # Returns: "first response"
puts fiber.resume("world")   # Prints: Received: world
                              # Returns: "final"
```

## Fiber vs Thread

| Aspect | Thread | Fiber |
|--------|--------|-------|
| Scheduling | Preemptive (OS) | Cooperative (you) |
| Overhead | ~1MB stack | ~4KB initial |
| Context switch | Expensive | Cheap |
| Parallelism | Yes (limited by GVL) | No (single thread) |

## Generator Pattern

```ruby
def fibonacci
  Fiber.new do
    a, b = 0, 1
    loop do
      Fiber.yield a
      a, b = b, a + b
    end
  end
end

fib = fibonacci
10.times { puts fib.resume }  # 0, 1, 1, 2, 3, 5, 8, 13, 21, 34
```

## Resources

- [Fiber Class](https://docs.ruby-lang.org/en/4.0/Fiber.html) — Fiber documentation

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*