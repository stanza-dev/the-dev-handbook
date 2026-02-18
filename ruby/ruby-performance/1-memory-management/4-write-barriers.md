---
source_course: "ruby-performance"
source_lesson: "ruby-performance-write-barriers"
---

# Write Barriers

Write barriers are a GC optimization that tracks when old objects reference young objects.

## Why Write Barriers Matter

Without write barriers, a minor GC would need to scan ALL objects to find young object references. Write barriers mark objects that need re-scanning.

```ruby
# When old_object.field = young_object happens,
# a write barrier marks old_object for scanning
```

## Ruby 3.3+ Improvements

Many core classes gained write barrier support:
- Array, Hash, Struct
- T_DATA objects
- Generic instance variable containers

This means fewer major GCs needed!

## Variable Width Allocation (VWA)

Ruby 3.2+ allocates objects of different sizes more efficiently:

```ruby
# Small objects share heap pages by size class
# Reduces fragmentation and improves cache locality

# Check if VWA is enabled
GC::INTERNAL_CONSTANTS[:SIZE_POOL_COUNT]  # > 1 means VWA is active
```

## Compaction

Ruby 3.0+ supports heap compaction:

```ruby
# Compact the heap (move objects to reduce fragmentation)
GC.compact

# Enable auto-compaction
GC.auto_compact = true

# Check compaction stats
GC.stat[:compact_count]
```

## Tips for GC-Friendly Code

1. **Reuse objects** when possible
2. **Freeze constants** to prevent duplication
3. **Use symbols** for enum-like values
4. **Avoid creating objects in hot loops**

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*