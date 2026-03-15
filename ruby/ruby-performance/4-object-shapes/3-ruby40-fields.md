---
source_course: "ruby-performance"
source_lesson: "ruby-performance-ruby40-fields"
---

# Ruby 4.0 Generic Ivar Fields

## Introduction
Ruby 3.2 introduced Object Shapes for standard Ruby objects, but objects backed by C extensions or special layouts stored ivars in a shared global table. Ruby 4.0 replaces that global table with per-instance "fields" structures, removing a major bottleneck for Ractor-based concurrency.

## Key Concepts
- **Generic ivar storage**: The fallback mechanism for objects that cannot embed ivars directly (e.g., objects with C-level `T_DATA` layouts). Before Ruby 4.0, these used a single global hash table protected by a lock.
- **Per-instance fields**: Ruby 4.0 gives each such object its own fields structure, eliminating lock contention when multiple Ractors access different objects.
- **Ractor**: Ruby's actor-based concurrency primitive. Each Ractor runs in its own thread with isolated memory, but shared data structures (like the old global ivar table) created contention points.

## Real World Context
If you run a Ractor-per-core architecture (common in Ruby 4.0 for CPU-bound workloads), the old global ivar table was a serialization bottleneck. With per-instance fields, each Ractor's objects are fully independent, and `Class#new` with keyword arguments runs significantly faster because there is no global lock to acquire.

## Deep Dive
In Ruby 3.x, generic ivars lived in a process-wide hash table:

```ruby
# Ruby 3.x internal pseudo-code:
# GLOBAL_GENERIC_IVAR_TABLE[object_id][:@name] = value
# Every read/write acquires a mutex -> contention under Ractors
```

Ruby 4.0 moves to per-instance storage:

```ruby
# Ruby 4.0 internal pseudo-code:
# object.fields[:@name] = value
# No global lock needed — each object owns its fields
```

The practical benefits for your code are:

- **Better Ractor scaling**: No contention on shared data structures when different Ractors touch different objects.
- **Faster `Class#new`**: Especially with keyword arguments, because field allocation is local to the new object.
- **Improved cache locality**: Per-object fields stay close in memory to the object header, reducing CPU cache misses.

You can verify that shapes work correctly using the debug API:

```ruby
obj = MyClass.new(name: "test")
shape = RubyVM.debug_get_shape(obj)
puts shape  # Shows the shape descriptor for this object
```

To maximize the benefit, follow the same rules as regular shapes — initialize all ivars upfront and in consistent order:

```ruby
class Optimized
  def initialize(a, b, c)
    @a = a
    @b = b
    @c = c
  end
end
```

## Common Pitfalls
1. **Assuming per-instance fields fix all Ractor issues** — Fields eliminate *ivar table* contention, but sharing mutable objects across Ractors is still restricted. You must still use `Ractor.make_shareable` or send copies.
2. **Ignoring the shape depth limit** — Ruby imposes a maximum shape tree depth. Dynamically creating many ivars via `instance_variable_set("@#{name}", val)` can exceed this limit, causing the VM to fall back to the slow generic hash.

## Best Practices
1. **Avoid dynamic ivar creation** — Never loop over data and call `instance_variable_set` with computed names. Use a Hash ivar instead: `@data[name] = val`.
2. **Benchmark with Ractors enabled** — The per-instance fields improvement is most visible under Ractor concurrency. Single-threaded benchmarks may not show the difference.

## Summary
- Ruby 4.0 replaces the global generic ivar table with per-instance fields, eliminating a Ractor contention bottleneck.
- `Class#new` with kwargs is faster, and cache locality improves because fields are co-located with the object.
- The shape depth limit still applies — avoid dynamic ivar creation to stay on the fast path.

## Code Examples

**Creating objects inside a Ractor benefits from per-instance fields in Ruby 4.0 — no global lock contention during ivar initialization.**

```ruby
# Ruby 4.0: Ractor-safe object with per-instance fields
ractor = Ractor.new do
  # Each Ractor creates objects independently — no global lock contention
  1000.times.map do |i|
    Config.new(host: "server-#{i}", port: 8080 + i, timeout: 30)
  end
end

results = ractor.take
puts results.size  # => 1000
# In Ruby 3.x, this would contend on the global ivar table
# In Ruby 4.0, each Config's fields are per-instance — zero contention
```


## Resources

- [Ractor Documentation](https://docs.ruby-lang.org/en/4.0/Ractor.html) — Official Ractor documentation covering actor-based concurrency in Ruby 4.0

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*