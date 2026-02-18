---
source_course: "ruby-performance"
source_lesson: "ruby-performance-ruby40-fields"
---

# Ruby 4.0: Per-Instance Fields

Ruby 4.0 improves instance variable handling for objects that don't embed ivars directly.

## The Problem: Generic Ivars

Some objects (like those with C extensions or special layouts) store ivars in a global table:

```ruby
# Before Ruby 4.0: Generic ivars used a global hash table
# High contention in multi-Ractor scenarios
```

## Ruby 4.0 Solution: Per-Instance Fields

```ruby
# Ruby 4.0: Each object gets its own "fields" object
# - Reduced lock contention
# - Better cache locality
# - Faster ivar access for generic objects
```

## Impact on Performance

- **Better Ractor scaling**: Less contention on shared data structures
- **Faster Class#new**: Especially with keyword arguments
- **Improved cache behavior**: Per-object fields stay close in memory

## Best Practices for Ruby 4.0

1. **Initialize all ivars in `initialize`**
2. **Use consistent ivar order across instances**
3. **Prefer embedded ivars** (standard Ruby objects)
4. **Minimize dynamic ivar creation**

```ruby
# Ideal for shapes: all ivars declared upfront
class Optimized
  def initialize(a, b, c)
    @a = a
    @b = b
    @c = c
  end
end
```

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*