---
source_course: "ruby-concurrency"
source_lesson: "ruby-concurrency-fibers-intro"
---

# Fibers: Cooperative Concurrency

## Introduction
Fibers are Ruby's primitive for cooperative (non-preemptive) concurrency. Unlike threads where the OS decides when to switch, with fibers YOU explicitly control when execution pauses and resumes. This makes fibers predictable, lightweight, and ideal for generators, coroutines, and async I/O patterns.

## Key Concepts
- **Fiber**: A lightweight execution context with its own call stack (~4KB initial) that you manually pause and resume.
- **Cooperative Scheduling**: The running fiber decides when to yield control — no preemption, no race conditions from unexpected switches.
- **Fiber.yield**: Pauses the current fiber and returns a value to whichever code called `resume`.

## Real World Context
The `async` gem (used by Falcon web server) builds on fibers to handle thousands of concurrent connections with minimal memory. Enumerator, the backbone of Ruby's lazy iteration, is implemented with fibers internally. Every time you use `each_with_object` or `Enumerator::Lazy`, fibers are working behind the scenes.

## Deep Dive

### Basic Fiber Usage

A fiber is created with `Fiber.new` and started with `resume`. When the fiber calls `Fiber.yield`, it pauses and returns the yielded value to the caller.

```ruby
report_fiber = Fiber.new do
  puts "Generating report..."
  Fiber.yield "Phase 1 complete"  # Pauses here, returns to caller
  puts "Finalizing report..."
  "Report done"
end

status = report_fiber.resume  # Prints: Generating report...
puts status                    # => "Phase 1 complete"

final = report_fiber.resume   # Prints: Finalizing report...
puts final                     # => "Report done"
```

The first `resume` runs the fiber until it hits `Fiber.yield`, which returns `"Phase 1 complete"` to the caller. The second `resume` continues from where the fiber paused and returns the block's final value.

### Two-Way Communication

Fibers support bidirectional data passing. The first `resume` passes its argument as the block parameter. Subsequent `resume` calls pass their argument as the return value of `Fiber.yield`.

```ruby
processor = Fiber.new do |initial_batch|
  puts "Processing: #{initial_batch}"
  next_batch = Fiber.yield "batch 1 done"
  puts "Processing: #{next_batch}"
  "all batches complete"
end

result1 = processor.resume("orders-jan")   # Prints: Processing: orders-jan
puts result1                                # => "batch 1 done"

result2 = processor.resume("orders-feb")   # Prints: Processing: orders-feb
puts result2                                # => "all batches complete"
```

The first resume delivers `"orders-jan"` as the `initial_batch` parameter. The second resume delivers `"orders-feb"` as the return value of `Fiber.yield`.

### Fiber vs Thread Comparison

| Aspect | Thread | Fiber |
|--------|--------|-------|
| Scheduling | Preemptive (OS decides) | Cooperative (you decide) |
| Stack size | ~1MB per thread | ~4KB initial per fiber |
| Context switch | Expensive (OS level) | Cheap (user-space) |
| Parallelism | Yes (limited by GVL) | No (single thread) |
| Race conditions | Possible | Impossible (no preemption) |

### Generator Pattern

Fibers are perfect for infinite sequences. The fiber yields values one at a time, pausing between each.

```ruby
def prime_generator
  Fiber.new do
    candidate = 2
    loop do
      if (2..Math.sqrt(candidate)).none? { |d| candidate % d == 0 }
        Fiber.yield candidate
      end
      candidate += 1
    end
  end
end

primes = prime_generator
10.times { print "#{primes.resume} " }
# Output: 2 3 5 7 11 13 17 19 23 29
```

The generator maintains its position between calls. Each `resume` produces the next prime without recomputing from scratch.

## Common Pitfalls
1. **Resuming a dead fiber** — Once a fiber's block finishes, calling `resume` raises `FiberError: dead fiber called`. Always check `fiber.alive?` before resuming if the number of yields is unknown.
2. **Confusing Fiber.yield with block yield** — `Fiber.yield` pauses the fiber and returns to the caller. Ruby's `yield` keyword calls a block. They are completely different mechanisms despite sharing the name.

## Best Practices
1. **Use fibers for lazy producers** — When you need to generate values on demand without computing the entire sequence upfront, fibers are more memory-efficient than building an array.
2. **Prefer Enumerator for simple generators** — Ruby's `Enumerator.new { |y| ... }` uses fibers internally and integrates with `each`, `map`, and other Enumerable methods. Use raw fibers only when you need two-way communication or custom scheduling.

## Summary
- Fibers are lightweight (~4KB) execution contexts with cooperative scheduling — you control when they pause and resume.
- `Fiber.yield` pauses a fiber and sends a value back to the caller; `resume` continues from where it paused.
- Two-way communication lets callers send data into fibers on each resume.
- Fibers cannot run in parallel but eliminate race conditions because switching is explicit.

## Code Examples

**An infinite Fibonacci sequence as a Fiber generator — each resume computes and yields only the next value, using no extra memory**

```ruby
# Infinite Fibonacci generator using a Fiber
def fibonacci
  Fiber.new do
    current, next_val = 0, 1
    loop do
      Fiber.yield current
      current, next_val = next_val, current + next_val
    end
  end
end

fib = fibonacci
10.times { print "#{fib.resume} " }
# Output: 0 1 1 2 3 5 8 13 21 34
```


## Resources

- [Fiber Class](https://docs.ruby-lang.org/en/4.0/Fiber.html) — Official Ruby Fiber class reference covering resume, yield, alive?, and scheduling

---

> 📘 *This lesson is part of the [Concurrency in Modern Ruby](https://stanza.dev/courses/ruby-concurrency) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*