---
source_course: "ruby-performance"
source_lesson: "ruby-performance-shape-polymorphism"
---

# Anti-Patterns for Object Shapes

## Problem: Different Initialization Orders

```ruby
# BAD - creates different shape trees
class User
  def initialize(role)
    if role == :admin
      @admin = true
      @name = "Admin"  # @admin first, then @name
    else
      @name = "User"   # @name first (different order!)
    end
  end
end
```

## Solution: Consistent Initialization

```ruby
# GOOD - same order always
class User
  def initialize(role)
    @name = nil
    @admin = nil

    if role == :admin
      @admin = true
      @name = "Admin"
    else
      @name = "User"
    end
  end
end
```

## Problem: Lazy Instance Variables

```ruby
# BAD - different shapes depending on method call order
class Cache
  def get(key)
    @cache ||= {}     # Only defined on first get!
    @cache[key]
  end

  def stats
    @stats ||= []     # Only defined on first stats!
    @stats
  end
end

c1 = Cache.new; c1.get(:x)   # Shape: root -> @cache
c2 = Cache.new; c2.stats      # Shape: root -> @stats (different!)
```

## Solution: Initialize Everything

```ruby
# GOOD - all ivars defined upfront
class Cache
  def initialize
    @cache = {}
    @stats = []
  end
  # ...
end
```

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*