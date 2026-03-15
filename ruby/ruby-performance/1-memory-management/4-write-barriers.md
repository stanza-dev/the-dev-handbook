---
source_course: "ruby-performance"
source_lesson: "ruby-performance-write-barriers"
---

# Write Barriers and GC Improvements in Ruby 4.0

## Introduction
Write barriers are a GC optimization that tracks when old-generation objects reference young-generation objects. Without them, every minor GC would need to scan the entire heap. Ruby 4.0 extends write-barrier protection to more core classes and introduces several GC improvements that reduce memory usage and pause times.

## Key Concepts
- **Write barrier**: A hook that fires when an old object stores a reference to a young object, marking the old object for re-scanning during the next minor GC.
- **Write-barrier-protected object**: An object whose class implements write barriers, enabling it to be safely skipped during minor GC unless flagged.
- **Heap size pools**: Ruby allocates objects into pools grouped by size class; in Ruby 4.0 these pools grow independently.
- **Heap compaction**: The process of physically moving objects to eliminate fragmentation and improve cache locality.

## Real World Context
Every class that gains write-barrier support reduces the number of major GCs your application triggers. In a Rails app handling thousands of requests per second, fewer major GCs translate directly to lower p99 latency.

## Deep Dive

### How Write Barriers Work

When an old object gains a reference to a young object, the write barrier marks the old object so the next minor GC knows to re-scan it:

```ruby
# Internally, when old_array << young_object happens,
# the write barrier marks old_array for scanning.
# Without this, minor GC would miss the young_object reference.
```

This mechanism is what makes generational GC safe: the collector can skip most old objects during minor GC, only re-scanning those flagged by write barriers.

### Ruby 4.0 Write-Barrier Additions

Ruby 4.0 adds write-barrier protection to several previously unprotected core classes:

```ruby
# Newly write-barrier-protected in Ruby 4.0:
# - Random
# - Enumerator::Product
# - Enumerator::Chain
# - Addrinfo
# - StringScanner
#
# This means minor GC can skip instances of these classes
# unless they are flagged, reducing major GC frequency.
```

Each addition means fewer objects need full scanning during minor GC, which shortens pause times.

### Independent Heap Size Pool Growth

In Ruby 4.0, heaps with different size pools grow independently rather than in lockstep. This means a pool for 40-byte objects can expand without forcing the 80-byte pool to grow too:

```ruby
# Ruby 4.0: each size pool grows based on its own demand
# Result: lower overall memory footprint for apps with
# uneven object-size distributions
GC::INTERNAL_CONSTANTS[:SIZE_POOL_COUNT]  # Number of size pools
```

This reduces wasted memory in applications that allocate many objects of one size but few of another.

### Faster Sweeping and Object IDs

Ruby 4.0 introduces faster sweeping on large object pages, reducing the time spent reclaiming memory after the mark phase. Additionally, `object_id` and `hash` calls on `Class` and `Module` objects are faster because they no longer require expensive lookups. The GC also avoids maintaining the `id2ref` table until the first call to `ObjectSpace._id2ref`, saving memory in the common case.

### Heap Compaction

Ruby supports heap compaction to reduce fragmentation:

```ruby
# Compact the heap (move objects to eliminate gaps)
GC.compact

# Enable automatic compaction after every major GC
GC.auto_compact = true

# Check compaction statistics
GC.stat[:compact_count]
```

Compaction moves objects so that free slots are consolidated, improving cache locality and reducing memory waste from fragmentation.

## Common Pitfalls
1. **Assuming all objects are write-barrier-protected** — C extensions and some core classes may lack write barriers, which forces the GC to treat them conservatively and scan them during every minor GC.
2. **Enabling auto_compact without benchmarking** — Compaction itself has a cost. In latency-sensitive applications, measure whether the reduced fragmentation outweighs the compaction pause.

## Best Practices
1. **Reuse objects and freeze constants** — Fewer allocations mean less GC work. Frozen string literals (`# frozen_string_literal: true`) prevent duplicate String allocations.
2. **Prefer symbols for enum-like values** — Symbols are never garbage collected and avoid repeated allocation, making them ideal for fixed sets of identifiers.

## Summary
- Write barriers let minor GC skip old objects unless they reference young objects.
- Ruby 4.0 adds write-barrier protection to `Random`, `Enumerator::Product`, `Enumerator::Chain`, `Addrinfo`, and `StringScanner`.
- Heap size pools now grow independently, reducing memory waste.
- `object_id`/`hash` on Class/Module is faster; the `id2ref` table is deferred until first use.
- Heap compaction consolidates free slots to reduce fragmentation.

## Code Examples

**Enabling auto-compaction and checking how many compaction passes have run**

```ruby
# Enable automatic heap compaction and verify it runs
GC.auto_compact = true

# Force a major GC to trigger compaction
GC.start(full_mark: true)

puts "Compactions run: #{GC.stat[:compact_count]}"
puts "Size pools: #{GC::INTERNAL_CONSTANTS[:SIZE_POOL_COUNT]}"
```


## Resources

- [GC Module — Ruby 4.0](https://docs.ruby-lang.org/en/4.0/GC.html) — Official GC documentation for Ruby 4.0, including compaction and tuning

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*