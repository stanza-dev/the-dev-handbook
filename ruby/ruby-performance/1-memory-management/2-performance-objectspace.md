---
source_course: "ruby-performance"
source_lesson: "ruby-performance-objectspace"
---

# ObjectSpace Deep Dive

## Introduction
ObjectSpace allows you to iterate over every live object in the Ruby process, making it indispensable for debugging memory issues. It is powerful but expensive, so it should be used carefully outside of development and debugging contexts.

## Key Concepts
- **ObjectSpace.each_object**: Iterates over all live instances of a given class or module.
- **ObjectSpace.memsize_of**: Returns the approximate memory footprint of a single object in bytes.
- **ObjectSpace.count_objects**: Returns a hash of object counts grouped by internal type tag.
- **ObjectSpace.reachable_objects_from**: Lists all objects directly referenced by a given object.

## Real World Context
When your production Ruby process is leaking memory, standard profilers may not pinpoint which class is accumulating instances. ObjectSpace lets you count live objects by type and trace reference chains to find the root cause.

## Deep Dive
Start by counting objects of a specific type to verify your hypothesis about a leak:

```ruby
require 'objspace'

# Count all String objects currently alive
ObjectSpace.each_object(String).count

# Count instances of a custom class
class User; end
5.times { User.new }
ObjectSpace.each_object(User).count  # => 5

# Get object counts grouped by internal type
ObjectSpace.count_objects
# => {:TOTAL=>51548, :FREE=>234, :T_OBJECT=>1287, ...}
```

The `each_object` method returns an Enumerator, so you can chain `.count`, `.select`, or `.map` on it. The `count_objects` method is cheaper because it does not yield each object individually.

To trace what an object references, or measure its size:

```ruby
require 'objspace'

obj = "hello"

# What objects does this one reference?
ObjectSpace.reachable_objects_from(obj)

# Get memory size of a single object
ObjectSpace.memsize_of(obj)  # => 40 (bytes)

# Total memory consumed by all instances of String
ObjectSpace.memsize_of_all(String)
```

`memsize_of` returns the shallow size of an object (its own struct), not the transitive closure. Use `reachable_objects_from` to walk the reference graph manually.

In Ruby 4.0, `ObjectSpace._id2ref` is discouraged and its backing table is now deferred. The GC now avoids maintaining the `id2ref` table until the first call to `_id2ref`, which saves memory in the common case:

```ruby
# Deprecated in Ruby 4.0 — avoid using object IDs to retrieve objects
id = "hello".object_id
ObjectSpace._id2ref(id)  # Warning: deprecated
```

If your code relies on `_id2ref`, migrate to storing direct object references or using `WeakRef` instead.

## Common Pitfalls
1. **Using each_object in hot paths** — Iterating over all live objects is O(n) in heap size. Never call it inside request handling or tight loops.
2. **Confusing memsize_of with total retained memory** — `memsize_of` reports shallow size only. A Hash with 10,000 entries reports only its own struct size, not the size of its keys and values.

## Best Practices
1. **Use count_objects for quick health checks** — It is much cheaper than `each_object` and gives a high-level view of object type distribution.
2. **Combine with GC.start before measuring** — Force a collection before counting so you see only truly retained objects, not garbage awaiting collection.

## Summary
- ObjectSpace lets you iterate, count, and measure all live Ruby objects.
- `memsize_of` reports shallow size; use `reachable_objects_from` for reference tracing.
- `_id2ref` is discouraged in Ruby 4.0; the GC defers the `id2ref` table until first use.
- Always call `GC.start` before measuring to avoid counting unreachable objects.

## Code Examples

**Counting live String instances and their total memory after forcing a GC for accuracy**

```ruby
require 'objspace'

# Count live String objects and measure total memory
GC.start  # Collect garbage first for accurate counts
string_count = ObjectSpace.each_object(String).count
string_mem   = ObjectSpace.memsize_of_all(String)
puts "#{string_count} strings using #{string_mem} bytes"
```


## Resources

- [ObjectSpace](https://docs.ruby-lang.org/en/4.0/ObjectSpace.html) — ObjectSpace module

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*