---
source_course: "ruby-performance"
source_lesson: "ruby-performance-memory-leaks"
---

# Debugging Memory Leaks

## Introduction
Ruby's GC handles most cleanup automatically, but certain coding patterns prevent objects from being collected, causing memory to grow without bound. Learning to identify and fix these patterns is critical for long-running services like web servers and background job processors.

## Key Concepts
- **Memory leak**: Objects that remain reachable and accumulate over time, even though the application no longer needs them.
- **Retained memory**: Memory still referenced after a profiling block completes, indicating objects that outlive their intended scope.
- **LRU cache**: A cache that evicts the Least Recently Used entries when it reaches a size limit, preventing unbounded growth.

## Real World Context
A Rails app that leaks 1 MB per minute will consume 1.4 GB in a day. Without identifying the leak source, operators resort to periodic restarts, which mask the problem and introduce downtime.

## Deep Dive
The most common leak patterns fall into three categories.

**1. Global caches that grow forever.** A naive memoization hash never evicts entries:

```ruby
# BAD — cache grows unbounded
$cache = {}
def expensive_lookup(id)
  $cache[id] ||= calculate(id)
end

# GOOD — use an LRU cache with a size limit
require 'lru_redux'
$cache = LruRedux::Cache.new(1000)  # Max 1000 entries
```

The bounded cache guarantees that memory usage plateaus, regardless of how many unique IDs arrive.

**2. Closures holding references to large objects.** A lambda captures its surrounding scope, keeping objects alive:

```ruby
# BAD — closure keeps reference to large_data
large_data = load_huge_file
callback = -> { puts large_data.size }
# large_data cannot be GC'd while callback exists!

# GOOD — extract only what you need
size = large_data.size
callback = -> { puts size }
large_data = nil  # Now it can be collected
```

By extracting the needed value before creating the closure, you break the reference chain to the large object.

**3. Event handlers that are never unsubscribed.** Subscribers accumulate if there is no cleanup mechanism:

```ruby
# BAD — subscribers accumulate
class EventBus
  def initialize
    @subscribers = []
  end
  def subscribe(&block)
    @subscribers << block  # Never cleaned up!
  end
end

# GOOD — return an unsubscribe function
def subscribe(&block)
  @subscribers << block
  -> { @subscribers.delete(block) }
end
```

Returning an unsubscribe callable lets consumers clean up when they are done.

To pinpoint leaks, use the `memory_profiler` gem, which reports retained and allocated objects:

```ruby
require 'memory_profiler'

report = MemoryProfiler.report do
  1000.times { User.new(name: "test") }
end

report.pretty_print
```

The report shows retained objects by gem, file, and location, making it easy to trace accumulation back to specific code.

## Common Pitfalls
1. **Assuming the GC will catch everything** — The GC only collects unreachable objects. If a global or long-lived structure holds a reference, the object is reachable and will never be freed.
2. **Profiling without forcing GC first** — If you measure memory without calling `GC.start`, you count objects that are already dead but not yet swept, giving inflated numbers.

## Best Practices
1. **Audit global state regularly** — Search for `$global`, `@@class_var`, and class-level instance variables that accumulate data. Each one is a potential leak.
2. **Use WeakRef for caches when appropriate** — `WeakRef` allows the GC to collect the referenced object if no strong references remain, preventing cache-driven leaks.

## Summary
- Unbounded caches, closures capturing large objects, and un-removed event handlers are the top three leak patterns.
- The `memory_profiler` gem reports retained objects by source location.
- Always force `GC.start` before measuring to get accurate retention counts.
- Use LRU caches or `WeakRef` to bound memory in long-running processes.

## Code Examples

**Using memory_profiler to identify retained objects and trace them to source locations**

```ruby
require 'memory_profiler'

report = MemoryProfiler.report do
  users = []
  1000.times { users << User.new(name: "test") }
end

# Shows retained objects grouped by gem, file, and class
report.pretty_print(to_file: 'mem_report.txt')
```


## Resources

- [memory_profiler Gem](https://github.com/SamSaffron/memory_profiler) — Gem for tracking retained and allocated Ruby objects

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*