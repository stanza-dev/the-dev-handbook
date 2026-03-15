---
source_course: "ruby-performance"
source_lesson: "ruby-performance-garbage-collection-ruby"
---

# Garbage Collection

## Introduction
Ruby uses a Generational Garbage Collector based on the hypothesis that most objects die young. Understanding how the GC works is essential for writing memory-efficient Ruby applications and diagnosing performance problems caused by excessive collection pauses.

## Key Concepts
- **Minor GC**: A fast collection that only checks young (newly allocated) objects. Most objects die here.
- **Major GC**: A slower collection that examines all objects, including those promoted to the old generation.
- **Object Promotion**: Objects that survive several minor GC cycles are promoted to the old generation, where they are checked less frequently.
- **Heap Slots**: Fixed-size memory slots where Ruby stores individual objects.

## Real World Context
In production Rails applications, GC pauses can add tens of milliseconds to request latency. Knowing whether your app triggers excessive major GCs helps you tune memory settings and reduce tail latencies.

## Deep Dive
Ruby's generational GC divides objects into young and old generations. The following code shows how to force a collection and inspect GC statistics:

```ruby
# Force garbage collection
GC.start

# Get GC statistics
stats = GC.stat
puts stats[:count]          # Total GC runs
puts stats[:minor_gc_count]  # Fast young-object collections
puts stats[:major_gc_count]  # Full collections including old objects
puts stats[:heap_live_slots] # Currently occupied heap slots
```

The `GC.stat` hash gives you a snapshot of the collector's state. The `:count` key is the total number of collections, while `:minor_gc_count` and `:major_gc_count` break that total down.

Objects survive promotion after a configurable number of minor GC cycles. You can inspect the promotion threshold:

```ruby
# Objects surviving 3 GC cycles become old
GC::INTERNAL_CONSTANTS[:RVALUE_OLD_AGE]  # => 3
```

This means an object must survive three minor GCs before Ruby considers it long-lived and promotes it.

You can also tune GC behavior through environment variables to match your workload:

```bash
# Increase initial heap size (reduces early GC frequency)
RUBY_GC_HEAP_INIT_SLOTS=600000

# Increase growth factor (larger heap expansions)
RUBY_GC_HEAP_GROWTH_FACTOR=1.25

# Set old-malloc limit before triggering a major GC
RUBY_GC_OLDMALLOC_LIMIT=64000000
```

These variables let you trade memory for fewer GC pauses, which is common in memory-rich server environments.

## Common Pitfalls
1. **Over-tuning GC environment variables** — Increasing heap size too aggressively wastes memory without improving throughput. Always benchmark before and after changes.
2. **Calling GC.start in production** — Forcing a full GC during request handling introduces unpredictable latency spikes. Let the runtime manage collection timing.

## Best Practices
1. **Monitor GC stats in production** — Instrument `GC.stat` in your metrics pipeline to spot trends in major GC frequency over time.
2. **Use RUBY_GC_HEAP_INIT_SLOTS for known workloads** — If your app stabilizes at 500k live objects, pre-allocating avoids dozens of early heap expansions.

## Summary
- Ruby uses a generational GC with minor (young-only) and major (full) collections.
- Objects are promoted to the old generation after surviving three minor GCs by default.
- `GC.stat` provides runtime insight into collection counts and heap usage.
- Environment variables let you tune heap size and growth to reduce GC frequency.

## Code Examples

**Reading GC statistics to monitor collection frequency and heap occupancy**

```ruby
# Inspect GC state to understand collection behavior
stats = GC.stat
puts "Total GC runs: #{stats[:count]}"
puts "Minor GCs:     #{stats[:minor_gc_count]}"
puts "Major GCs:     #{stats[:major_gc_count]}"
puts "Live slots:    #{stats[:heap_live_slots]}"
```


## Resources

- [GC Module](https://docs.ruby-lang.org/en/4.0/GC.html) — GC module documentation

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*