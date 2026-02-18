---
source_course: "ruby-performance"
source_lesson: "ruby-performance-understanding-object-shapes"
---

# Object Shapes

Introduced in Ruby 3.2, Object Shapes optimize instance variable access by tracking the "shape" of objects.

## How Shapes Work

Each time you define an instance variable (`@x`), Ruby transitions the object to a new shape. Objects with the same variables defined in the same order share a shape.

```ruby
class Point
  def initialize(x, y)
    @x = x  # Transitions: root -> shape_x
    @y = y  # Transitions: shape_x -> shape_xy
  end
end

p1 = Point.new(1, 2)  # Shape: shape_xy
p2 = Point.new(3, 4)  # Shape: shape_xy (same!)
```

## Shape Trees

```
                    root
                     |
                  shape_x (@x)
                   /    \
          shape_xy      shape_xz
          (@x, @y)      (@x, @z)
```

## Why Shapes Matter

With consistent shapes:
- **Fast ivar lookup**: Direct slot index instead of hash lookup
- **Better JIT optimization**: Type-stable access patterns
- **Memory efficiency**: Shared shape metadata

## Checking Object Shape

```ruby
# Ruby internals (debug only)
RubyVM.debug_get_shape(obj)
```

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*