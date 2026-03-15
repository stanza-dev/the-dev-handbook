---
source_course: "ruby-performance"
source_lesson: "ruby-performance-shape-debugging"
---

# Debugging Shape Issues

## Introduction
Shape problems are invisible at the Ruby source level — your code works correctly, just slower. Detecting shape mismatches requires specific tools and techniques to surface what the VM is doing under the hood.

## Key Concepts
- **`RubyVM.debug_get_shape`**: A debug API that returns the shape descriptor of an object, allowing you to compare shapes between instances.
- **Shape depth limit**: Ruby imposes a maximum depth for the shape tree. Objects that exceed this limit (by dynamically creating too many unique ivars) fall back to a slower hash-based storage.
- **YJIT side exits**: When YJIT encounters an unexpected shape at a cached ivar access site, it takes a "side exit" back to the interpreter, negating the JIT speedup.

## Real World Context
A major e-commerce platform discovered that their product catalog objects had 47 different shapes due to optional attributes added by different middleware layers. After unifying initialization order, ivar access throughput improved by 22% and YJIT side exit count dropped by 60%.

## Deep Dive
The primary debugging tool is `RubyVM.debug_get_shape`, which lets you compare two objects:

```ruby
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

Even though both users have `@name` and `@age`, the different assignment order creates different shapes. The fix is to use a constructor:

```ruby
class User
  attr_accessor :name, :age

  def initialize(name, age)
    @name = name  # Always first
    @age  = age   # Always second
  end
end

u1 = User.new("A", 1)
u2 = User.new("B", 2)
same_shape?(u1, u2)  # => true
```

You can also monitor YJIT for shape-related deoptimization:

```ruby
stats = RubyVM::YJIT.runtime_stats
puts stats[:side_exit_count]  # High numbers may indicate shape issues
puts stats[:total_exits]
```

The shape depth limit is important to know about. When you dynamically create ivars, each unique name adds a level to the shape tree:

```ruby
# BAD — each key creates a new shape transition
data.each { |k, v| instance_variable_set("@#{k}", v) }
# If data has 100 keys, this creates a chain 100 levels deep
# Exceeding the limit causes fallback to hash-based storage

# GOOD — store dynamic data in a Hash ivar
def initialize(data)
  @data = data  # One shape transition, regardless of data size
end
```

## Common Pitfalls
1. **Relying on `debug_get_shape` in production** — This method is for debugging only. It may not be available in all Ruby builds and adds overhead. Use it in tests and development, not in production code.
2. **Ignoring the shape limit for serialization** — Libraries that deserialize JSON into ivars via `instance_variable_set` can easily hit the shape depth limit with large payloads.

## Best Practices
1. **Add shape assertions to your test suite** — Write a test that creates instances via different code paths and asserts they share the same shape (or at least the same `instance_variables` list).
2. **Monitor YJIT side exits after deploys** — A spike in `side_exit_count` after a deploy can indicate that new code introduced shape polymorphism.

## Summary
- Use `RubyVM.debug_get_shape` to verify that instances share the same shape.
- Dynamic ivar creation via `instance_variable_set` risks hitting the shape depth limit.
- Monitor YJIT `side_exit_count` to detect shape-related deoptimization in production.

## Code Examples

**Comparing dynamic ivar creation (hits shape limit) vs storing data in a Hash ivar (one shape transition regardless of attribute count).**

```ruby
# BAD: Dynamic ivar creation hits shape depth limit
class DynamicModel
  def initialize(attrs)
    attrs.each { |k, v| instance_variable_set("@#{k}", v) }
  end
end
# DynamicModel.new(a: 1, b: 2, ..., z: 26) -> 26-deep shape chain

# GOOD: Store dynamic attributes in a hash
class SafeModel
  def initialize(attrs)
    @attributes = attrs  # Single shape transition
  end

  def [](key)
    @attributes[key]
  end
end
```


## Resources

- [YJIT Documentation](https://docs.ruby-lang.org/en/4.0/jit/yjit_md.html) — Official YJIT docs covering runtime stats and shape-related side exits

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*