---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-open-classes"
---

# Open Classes

In Ruby, class definitions are active code execution. You can reopen any class, even standard ones, to add or modify methods.

```ruby
class String
  def shout
    upcase + "!"
  end
end

"hello".shout # => "HELLO!"
```

## How It Works

The `class` keyword doesn't just define a class - it opens the class for modification:

```ruby
# First definition
class User
  def name
    @name
  end
end

# Later, reopen and add method
class User
  def email
    @email
  end
end

# User now has both methods!
```

## Monkey Patching

Modifying existing classes (especially core classes) is called "monkey patching." It's powerful but dangerous:

```ruby
# Dangerous: modifying core behavior
class Array
  def first
    "surprise!"  # Breaks everything!
  end
end
```

## When to Use Open Classes

- **DSLs**: Adding domain-specific methods
- **Extensions**: Adding missing functionality to gems
- **Polyfills**: Backporting newer Ruby features
- **Testing**: Adding test helpers

**Better alternative**: Use Refinements for scoped modifications.

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*