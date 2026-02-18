---
source_course: "ruby-performance"
source_lesson: "ruby-performance-garbage-collection-ruby"
---

# Generational GC

Ruby uses a Generational Garbage Collector based on the hypothesis that most objects die young.

## Two Types of Collection

- **Minor GC**: Fast, only checks young objects. Most objects die here.
- **Major GC**: Slow, checks everything including old (promoted) objects.

```ruby
# Force garbage collection
GC.start

# Get GC statistics
stats = GC.stat
puts stats[:count]        # Total GC runs
puts stats[:minor_gc_count]
puts stats[:major_gc_count]
puts stats[:heap_live_slots]
```

## Object Promotion

Objects that survive multiple minor GCs are "promoted" to old generation:

```ruby
# Objects surviving 3 GC cycles become old
GC::INTERNAL_CONSTANTS[:RVALUE_OLD_AGE]  # => 3
```

## GC Tuning Environment Variables

```bash
# Increase initial heap size
RUBY_GC_HEAP_INIT_SLOTS=600000

# Increase growth factor
RUBY_GC_HEAP_GROWTH_FACTOR=1.25

# Set old object limit before major GC
RUBY_GC_OLDMALLOC_LIMIT=64000000
```

## Resources

- [GC Module](https://docs.ruby-lang.org/en/4.0/GC.html) — GC module documentation

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*