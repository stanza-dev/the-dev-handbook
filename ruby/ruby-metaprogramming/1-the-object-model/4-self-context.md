---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-self-context"
---

# Understanding Self

## Introduction

The `self` keyword is the compass of Ruby metaprogramming. It always refers to the current object, but which object that is changes depending on where you are in the code. Mastering `self` is the key to understanding method definition, variable access, and scope.

## Key Concepts

- **self**: A reference to the current object that Ruby uses as the default receiver for method calls.
- **Default receiver**: When you call a method without an explicit receiver, Ruby sends it to `self`.
- **Class-level self**: Inside a class body but outside any method, `self` is the class object itself.

## Real World Context

A common bug in Ruby is writing `name = name` in an initializer instead of `self.name = name`. The first version creates a local variable; the second calls the setter method. Understanding what `self` refers to in each context prevents this class of bugs and is essential for writing DSLs where method calls implicitly target `self`.

## Deep Dive

`self` always refers to the current object, but what that means changes based on context. The following example maps out every major context:

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

At the top level, `self` is the special `main` object. Inside a class body, `self` is the class itself. Inside an instance method, `self` is the instance that received the call.

When you call a method without an explicit receiver, Ruby uses `self` as the receiver:

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

In `initialize`, `self.name = name` explicitly sends the `name=` message to `self`. Without `self.`, Ruby would interpret `name = name` as a local variable assignment, which is a common source of bugs.

At the class level, `self` refers to the class object, which is why `class << self` opens the singleton class of the class:

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

Both `self.debug_mode = false` in the class body and `self.debug_mode = true` inside the class method refer to the `Configuration` class object.

## Common Pitfalls

1. **Local variable shadowing setters** — Writing `name = value` instead of `self.name = value` creates a local variable and never calls the setter. This is the most common `self`-related bug.
2. **Assuming self is the same in blocks** — In most blocks, `self` stays the same as the surrounding context. However, `instance_eval` and `class_eval` change `self`, which can be surprising.

## Best Practices

1. **Always use `self.` for setter calls** — Even though getter calls work without `self.`, always use `self.attr = value` for setters to avoid shadowing.
2. **Print `self` when debugging** — When metaprogramming gets confusing, insert `puts self.inspect` to see exactly which object is the current receiver.

## Summary

- `self` is the current object and changes depending on context: top-level, class body, instance method, or class method.
- Methods called without an explicit receiver are sent to `self`.
- Understanding `self` is critical for writing correct setters, defining class methods, and building DSLs.

## Code Examples

**Demonstrating why self is required for setter calls but optional for getter calls**

```ruby
class Logger
  attr_accessor :level

  def initialize(level)
    self.level = level  # Calls setter — correct
    # level = level     # Local variable — bug!
  end

  def info?
    level == :info  # Calls getter — self. is optional for reads
  end
end

logger = Logger.new(:info)
logger.info? # => true
```


## Resources

- [Modules and Classes Syntax](https://docs.ruby-lang.org/en/4.0/syntax/modules_and_classes_rdoc.html) — Official Ruby documentation covering self in module and class contexts.

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*