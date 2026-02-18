---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-self-context"
---

# The self Keyword

`self` always refers to the "current object" - but what that means changes based on context.

## Self in Different Contexts

```ruby
# At top level
self  # => main (an Object)

# Inside a class definition
class User
  self  # => User (the class)

  def instance_method
    self  # => the User instance
  end

  def self.class_method
    self  # => User (the class)
  end
end

# Inside a module
module Helper
  self  # => Helper
end
```

## Self and Method Calls

When you call a method without an explicit receiver, Ruby uses `self`:

```ruby
class Person
  attr_accessor :name

  def initialize(name)
    self.name = name    # Calls name= on self
    # name = name       # Would just assign local variable!
  end

  def greet
    introduce           # Same as self.introduce
  end

  private

  def introduce
    "I'm #{name}"
  end
end
```

## Class-Level Self

```ruby
class Configuration
  class << self
    attr_accessor :debug_mode
  end

  self.debug_mode = false  # self is Configuration

  def self.enable_debug
    self.debug_mode = true  # self is still Configuration
  end
end
```

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*