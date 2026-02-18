---
source_course: "ruby-performance"
source_lesson: "ruby-performance-allocation-profiling"
---

# Tracking Object Allocations

Excessive object allocation causes GC pressure. Finding allocation hotspots is critical for performance.

## Using allocation_tracer Gem

```ruby
require 'allocation_tracer'

AllocationTracer.trace do
  1000.times { "hello" + "world" }  # Creates strings
end

# Results by file:line
AllocationTracer.trace
  .sort_by { |k, v| -v[:count] }
  .first(10)
```

## Using StackProf Object Mode

```ruby
StackProf.run(mode: :object, out: 'alloc.dump') do
  allocation_heavy_code
end

# stackprof alloc.dump --text
```

## Common Allocation Hotspots

```ruby
# 1. String interpolation in loops
1000.times { |i| "Item #{i}" }  # 1000 strings!

# Better: use format once
items = (0...1000).map { |i| "Item #{i}" }

# 2. Implicit array creation
def method(*args)  # Creates array for every call
  args.each { |a| process(a) }
end

# 3. Hash literal in loops
1000.times { { key: value } }  # 1000 hashes!

# Better: reuse if possible
template = { key: nil }.freeze
```

## Reducing Allocations

1. **Reuse objects** when safe
2. **Freeze constants** for deduplication
3. **Use bang methods** that modify in place
4. **Buffer strings** with StringIO
5. **Pre-allocate arrays** with known sizes

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*