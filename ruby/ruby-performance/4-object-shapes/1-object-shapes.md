---
source_course: "ruby-performance"
source_lesson: "ruby-performance-understanding-object-shapes"
---

# Object Shapes (Shapes & Slots)

## Introduction
Before Ruby 3.2, every instance variable lookup required a hash table search inside the object. Object Shapes replace that hash lookup with a direct array index, making ivar access as fast as reading a struct field in C.

## Key Concepts
- **Object Shape**: A metadata descriptor that records which instance variables an object has and in what order they were defined. Two objects with the same ivars defined in the same order share one shape.
- **Shape transition**: When you assign a new instance variable, the object moves from its current shape to a child shape in a tree structure.
- **Slot index**: Because the shape knows exactly which ivar lives at which position, the VM can read `@x` by index (O(1)) instead of by hash lookup (amortized O(1) but with higher constant factor and cache misses).

## Real World Context
Rails models often have 10-20 instance variables. Before Object Shapes, every `@attributes` or `@association_cache` access paid the hash lookup cost. With shapes, these reads are a single pointer dereference, which compounds across millions of requests per day.

## Deep Dive
When you define instance variables in `initialize`, Ruby builds a shape tree:

```ruby
class Point
  def initialize(x, y)
    @x = x  # Transitions: root -> shape_with_x
    @y = y  # Transitions: shape_with_x -> shape_with_x_y
  end
end

p1 = Point.new(1, 2)  # Shape: shape_with_x_y
p2 = Point.new(3, 4)  # Shape: shape_with_x_y (same shape!)
```

Because `p1` and `p2` both defined `@x` then `@y` in that order, they share the same shape. The VM knows `@x` is at slot 0 and `@y` is at slot 1 for any object with this shape.

The shape tree branches when different objects define different ivars or define them in different order:

```
                    root
                     |
                  shape_x (@x at slot 0)
                   /    \
         shape_x_y       shape_x_z
     (@y at slot 1)     (@z at slot 1)
```

You can inspect an object's shape using a debug method:

```ruby
# Debug-only — not for production use
RubyVM.debug_get_shape(Point.new(1, 2))
```

This returns the shape object, which you can compare between instances to verify they match.

## Common Pitfalls
1. **Defining ivars in different orders** — If one code path sets `@a` then `@b`, and another sets `@b` then `@a`, the objects get different shapes even though they have the same variables. This defeats the optimization.
2. **Assuming shapes work across classes** — Shapes are per-class lineage. Two unrelated classes with identical ivar names still have separate shape trees.

## Best Practices
1. **Initialize all ivars in the constructor** — Defining every ivar in `initialize` guarantees all instances of that class share one shape from the start.
2. **Use a consistent definition order** — Even if you set ivars outside the constructor, always set them in the same order across all code paths.

## Summary
- Object Shapes (Ruby 3.2+) turn instance variable access from hash lookup to direct slot indexing.
- Objects sharing the same ivar definition order share one shape, enabling the JIT to cache slot offsets.
- Always initialize all instance variables upfront and in a consistent order.

## Code Examples

**Initializing all ivars to nil in the constructor ensures every Config instance has the same shape from creation.**

```ruby
class Config
  def initialize
    @host = nil
    @port = nil
    @timeout = nil
  end

  def load(host:, port:, timeout: 30)
    @host    = host
    @port    = port
    @timeout = timeout
  end
end

c1 = Config.new
c2 = Config.new
# Both share the same shape (root -> @host -> @port -> @timeout)
# Even before load() is called, ivars exist at known slots
```


## Resources

- [Object Shapes in CRuby](https://docs.ruby-lang.org/en/4.0/Object.html) — Ruby Object class documentation covering instance variable storage

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*