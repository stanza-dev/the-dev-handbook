---
source_course: "ruby"
source_lesson: "ruby-modules-mixins"
---

# Modules & Mixins

## Introduction
Ruby intentionally does not support multiple inheritance, but it provides something arguably more powerful: modules and mixins. By mixing modules into classes, you can share behavior across unrelated types without the complexity of a multi-inheritance hierarchy. This lesson covers `include`, `extend`, and `prepend` — the three ways to mix modules into classes.

## Key Concepts
- **Module**: A collection of methods and constants that cannot be instantiated. Used for namespacing and as mixins.
- **Mixin**: The act of adding a module's methods to a class via `include`, `extend`, or `prepend`.
- **`include`**: Adds module methods as instance methods of the class.
- **`extend`**: Adds module methods as class methods (singleton methods on the class object).
- **`prepend`**: Inserts the module before the class in the ancestor chain, allowing method interception.
- **Ancestor Chain**: The ordered list of classes and modules Ruby searches when resolving a method call.

## Real World Context
Mixins are everywhere in production Ruby. In Rails, `ActiveModel::Validations` is a module you include to get validation methods. A logging concern might be prepended to intercept `save` calls. Understanding the difference between `include`, `extend`, and `prepend` is critical when designing reusable behavior — for instance, an auditing module that logs every database write by prepending the `save` method.

## Deep Dive
Ruby does not support multiple inheritance. Instead, it uses modules for code reuse across unrelated classes.

### Include: Instance Methods

`include` adds module methods as instance methods on the class:

```ruby
module Walkable
  def walk
    puts "#{self.class} is walking"
  end
end

class Person
  include Walkable
end

class Robot
  include Walkable
end

Person.new.walk  # "Person is walking"
Robot.new.walk   # "Robot is walking"
```

Both `Person` and `Robot` gain the `walk` instance method without any inheritance relationship between them. The module appears in each class's ancestor chain.

### Extend: Class Methods

`extend` adds module methods as class methods (singleton methods on the class):

```ruby
module Findable
  def find(id)
    puts "Finding #{self} with id #{id}"
  end
end

class User
  extend Findable
end

User.find(1)  # "Finding User with id 1"
```

Here `find` is available on the `User` class itself, not on instances. This pattern is commonly used for factory methods and query interfaces.

### Prepend: Method Interception

`prepend` inserts the module *before* the class in the ancestor chain, which means the module's methods are called first:

```ruby
module Logging
  def save
    puts "About to save..."
    super              # Calls the original
    puts "Saved!"
  end
end

class Record
  prepend Logging

  def save
    puts "Saving record"
  end
end

Record.new.save
# About to save...
# Saving record
# Saved!
```

Because `Logging` is prepended, its `save` runs first. Calling `super` delegates to the class's own `save`. This is the cleanest way to wrap existing behavior without aliasing methods.

## Common Pitfalls
1. **Confusing `include` and `extend`** — `include` adds instance methods, `extend` adds class methods. Mixing them up means your methods end up in the wrong scope and you get `NoMethodError` at runtime.
2. **Not understanding ancestor chain order** — When multiple modules are included, the last one included is searched first. This can cause unexpected method resolution if two modules define the same method name.

## Best Practices
1. **Prefer `prepend` over `alias_method` for wrapping** — When you need to intercept an existing method, `prepend` gives a clean `super` chain. The old `alias_method_chain` pattern is fragile and deprecated.
2. **Keep modules focused on a single responsibility** — A module named `Walkable` should only contain walking-related methods. Mixing unrelated concerns into one module defeats the purpose of mixins.

## Summary
- `include` adds module methods as instance methods; `extend` adds them as class methods.
- `prepend` inserts the module before the class in the ancestor chain, enabling clean method interception via `super`.
- Modules let you share behavior across unrelated classes without the rigidity of multiple inheritance.

## Code Examples

**Using include for instance methods and extend for class methods from modules**

```ruby
module Walkable
  def walk
    puts "#{self.class} is walking"
  end
end

module Findable
  def find(id)
    puts "Finding #{self} with id #{id}"
  end
end

class Person
  include Walkable   # Instance methods
  extend Findable    # Class methods
end

Person.new.walk  # "Person is walking"
Person.find(1)   # "Finding Person with id 1"
```


## Resources

- [Module Class](https://docs.ruby-lang.org/en/4.0/Module.html) — Official Module documentation

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*