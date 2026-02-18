---
source_course: "ruby-performance"
source_lesson: "ruby-performance-shape-debugging"
---

# Identifying Shape Problems

## Signs of Shape Polymorphism

1. **Inconsistent performance** - Same code, varying speed
2. **JIT deoptimization** - Side exits in YJIT
3. **High memory for ivars** - More shapes = more metadata

## Using RubyVM Debug Methods

```ruby
# Check if two objects have the same shape
def same_shape?(a, b)
  RubyVM.debug_get_shape(a) == RubyVM.debug_get_shape(b)
end

class User
  attr_accessor :name, :age
end

u1 = User.new.tap { |u| u.name = "A"; u.age = 1 }
u2 = User.new.tap { |u| u.age = 2; u.name = "B" }  # Different order!

same_shape?(u1, u2)  # => false!
```

## Fixing Shape Issues

```ruby
# Use attr_accessor to ensure consistent order
class User
  attr_accessor :name, :age

  def initialize(name, age)
    @name = name  # Always first
    @age = age    # Always second
  end
end
```

## YJIT Shape Statistics

```ruby
# Check for shape-related deoptimization
stats = RubyVM::YJIT.runtime_stats

# High numbers here indicate shape issues
# (exact stat names may vary by Ruby version)
```

## Shape Limit

Ruby has a maximum shape tree depth. Hitting this limit causes fallback to slower hash-based ivars:

```ruby
# Avoid dynamically creating many ivars
# BAD:
data.each { |k, v| instance_variable_set("@#{k}", v) }
```

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*