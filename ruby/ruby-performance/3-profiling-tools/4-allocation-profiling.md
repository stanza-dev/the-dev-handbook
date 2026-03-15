---
source_course: "ruby-performance"
source_lesson: "ruby-performance-allocation-profiling"
---

# Allocation Profiling

## Introduction
Every object Ruby allocates eventually needs to be garbage collected. When your hot loop creates thousands of short-lived objects, GC pauses add up. Allocation profiling shows you exactly where those objects come from.

## Key Concepts
- **Allocation hotspot**: A line of code that creates a disproportionate number of objects during execution.
- **GC pressure**: The rate at which new allocations fill heap slots, forcing the garbage collector to run more frequently.
- **Retained vs allocated**: Allocated objects are everything created during profiling. Retained objects are those still alive at the end — these are the ones that survive GC and consume long-term memory.

## Real World Context
A Rails endpoint that allocates 50,000 objects per request will spend 10-15% of its time in GC. Reducing allocations by half can cut response times by 30ms or more — a meaningful improvement at scale.

## Deep Dive
The `memory_profiler` gem tracks every allocation and groups results by gem, file, and line:

```ruby
require 'memory_profiler'

report = MemoryProfiler.report do
  1000.times { User.new(name: "test") }
end

report.pretty_print
```

The output separates allocated objects (created during the block) from retained objects (still alive afterward). Focus on retained objects first, since they indicate potential memory leaks.

StackProf's `:object` mode is lighter-weight and works well in production:

```ruby
require 'stackprof'

StackProf.run(mode: :object, out: 'alloc.dump') do
  allocation_heavy_code
end
# stackprof alloc.dump --text
```

Common allocation hotspots include string interpolation in loops, implicit splat arrays, and hash literals created repeatedly:

```ruby
# BAD: 1000 string allocations
1000.times { |i| log("Processing item #{i}") }

# BETTER: only allocate when log level is enabled
1000.times { |i| log("Processing item #{i}") if logger.debug? }

# BAD: splat creates a new array on every call
def process(*args)
  args.each { |a| handle(a) }
end

# BAD: hash literal in a loop
1000.times { { key: compute_value } }

# BETTER: reuse a mutable hash if safe
buffer = {}
1000.times { buffer[:key] = compute_value; consume(buffer) }
```

## Common Pitfalls
1. **Optimizing allocated instead of retained** — Many allocated objects die immediately and are collected cheaply by minor GC. Focus on retained objects first, since they cause long-term memory growth.
2. **Freezing strings that are never duplicated** — `"constant".freeze` only helps when the same literal appears in a hot loop. When the `# frozen_string_literal: true` pragma is active in a file, individual freezes are redundant.

## Best Practices
1. **Use `memory_profiler` in development, StackProf `:object` in production** — `memory_profiler` instruments every allocation (high overhead), while StackProf samples (low overhead).
2. **Pre-allocate arrays with known sizes** — `Array.new(size)` avoids repeated resizing. For hashes, use `Hash.new` with a default rather than re-creating literals.

## Summary
- Excessive allocations increase GC frequency and pause time.
- Use `memory_profiler` for detailed allocation/retention reports and StackProf `:object` for sampling.
- Focus on retained objects for memory leaks and allocated objects in hot paths for latency.

## Code Examples

**Using memory_profiler to count allocations — each loop iteration creates a Hash and a String, all retained because the array holds references.**

```ruby
require 'memory_profiler'

report = MemoryProfiler.report do
  data = []
  1000.times do |i|
    data << { id: i, label: "Item #{i}" }  # 1000 Hashes + 1000 Strings
  end
end

report.pretty_print(to_file: 'mem_report.txt')
# Key output lines:
#   Total allocated: 2000 objects (1000 Hash, 1000 String)
#   Total retained:  2000 objects (all kept in `data` array)
```


## Resources

- [memory_profiler GitHub Repository](https://github.com/SamSaffron/memory_profiler) — Detailed memory profiling for Ruby with allocated vs retained breakdown

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*