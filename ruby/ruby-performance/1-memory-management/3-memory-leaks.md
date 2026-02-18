---
source_course: "ruby-performance"
source_lesson: "ruby-performance-memory-leaks"
---

# Common Causes of Memory Leaks

Ruby's GC handles most cleanup, but some patterns cause leaks:

## 1. Global Caches That Grow Forever

```ruby
# BAD - cache grows unbounded
$cache = {}
def expensive_lookup(id)
  $cache[id] ||= calculate(id)
end

# GOOD - use LRU cache with size limit
require 'lru_redux'
$cache = LruRedux::Cache.new(1000)  # Max 1000 entries
```

## 2. Closures Holding References

```ruby
# BAD - closure keeps reference to large_data
large_data = load_huge_file
callback = -> { puts large_data.size }
# large_data can't be GC'd while callback exists!

# GOOD - extract only what you need
size = large_data.size
callback = -> { puts size }
large_data = nil  # Now it can be collected
```

## 3. Event Handlers & Callbacks

```ruby
# BAD - subscribers accumulate
class EventBus
  def initialize
    @subscribers = []
  end

  def subscribe(&block)
    @subscribers << block  # Never cleaned up!
  end
end

# GOOD - provide unsubscribe
def subscribe(&block)
  @subscribers << block
  -> { @subscribers.delete(block) }  # Return unsubscribe function
end
```

## Using memory_profiler Gem

```ruby
require 'memory_profiler'

report = MemoryProfiler.report do
  # Your code here
  1000.times { User.new(name: "test") }
end

report.pretty_print
```

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*