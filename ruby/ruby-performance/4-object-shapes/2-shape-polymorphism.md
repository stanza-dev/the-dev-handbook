---
source_course: "ruby-performance"
source_lesson: "ruby-performance-shape-polymorphism"
---

# Avoiding Shape Polymorphism

## Introduction
Shape polymorphism occurs when instances of the same class end up with different shapes. This forces the JIT to generate slower, generic code paths instead of optimized direct slot reads. Understanding the anti-patterns helps you write shape-consistent code.

## Key Concepts
- **Shape polymorphism**: When different instances of the same class have different shapes due to conditional or lazy ivar definitions.
- **Monomorphic access**: When a given ivar access site always sees the same shape, allowing the JIT to compile a single fast path.
- **Megamorphic access**: When too many different shapes are seen at one access site, forcing the VM to fall back to a generic (slow) lookup.

## Real World Context
In large Rails codebases, plugins and concerns often add instance variables in `after_initialize` callbacks. If two plugins add ivars in different orders depending on configuration, every model instance can end up with a unique shape, silently degrading performance across the entire application.

## Deep Dive
The most common anti-pattern is conditional initialization:

```ruby
# BAD — creates different shapes based on role
class User
  def initialize(role)
    if role == :admin
      @admin = true
      @name = "Admin"  # Shape: root -> @admin -> @name
    else
      @name = "User"   # Shape: root -> @name (different!)
    end
  end
end
```

The fix is to always define all ivars in the same order, even if some start as `nil`:

```ruby
# GOOD — same shape for every instance
class User
  def initialize(role)
    @name  = nil
    @admin = nil

    if role == :admin
      @admin = true
      @name  = "Admin"
    else
      @name = "User"
    end
  end
end
```

Lazy instance variables are another frequent source of shape divergence:

```ruby
# BAD — shape depends on which method is called first
class Cache
  def get(key)
    @store ||= {}      # Only defined on first get!
    @store[key]
  end

  def stats
    @hits ||= 0         # Only defined on first stats!
    @hits
  end
end

c1 = Cache.new; c1.get(:x)    # Shape: root -> @store
c2 = Cache.new; c2.stats       # Shape: root -> @hits
c3 = Cache.new; c3.get(:x); c3.stats  # Shape: root -> @store -> @hits
# Three different shapes for the same class!
```

The fix is eager initialization:

```ruby
# GOOD — all instances get the same shape immediately
class Cache
  def initialize
    @store = {}
    @hits  = 0
  end
end
```

## Common Pitfalls
1. **Using `||=` for expensive defaults** — Developers use `@cache ||= expensive_setup` to defer work. Instead, call the setup in `initialize` or use a method that checks a flag without creating a new ivar.
2. **Mixing `attr_writer` with manual ivar sets** — If some code uses `self.name = x` (which calls the setter) and other code uses `@name = x`, the ivar is still `@name` in both cases, but if the setter adds validation ivars, shape order can diverge.

## Best Practices
1. **Define every ivar in `initialize`** — Even if the value is `nil`, `0`, or `[]`. The cost of initializing a nil is negligible compared to the cost of shape polymorphism.
2. **Audit ivars with `instance_variables`** — In tests, assert that all instances have the same `instance_variables` list to catch shape divergence early.

## Summary
- Conditional or lazy ivar definitions create shape polymorphism, slowing the JIT and ivar access.
- Always define all instance variables in the constructor, in a fixed order.
- Use `instance_variables` in tests to detect shape divergence before it reaches production.

## Code Examples

**A test that catches shape divergence by comparing instance_variables between two objects of the same class.**

```ruby
# Detecting shape divergence in tests
class ShapeTest < Minitest::Test
  def test_all_users_share_shape
    admin = User.new(:admin)
    guest = User.new(:guest)

    assert_equal admin.instance_variables.sort,
                 guest.instance_variables.sort,
                 "Shape divergence: admin and guest have different ivars"
  end
end
```


## Resources

- [Ruby Performance Tips — Instance Variables](https://docs.ruby-lang.org/en/4.0/syntax/assignment_rdoc.html) — Ruby assignment syntax documentation covering instance variable semantics

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*