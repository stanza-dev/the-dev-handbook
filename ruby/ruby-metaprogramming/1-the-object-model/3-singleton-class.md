---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-singleton-class"
---

# The Singleton Class

## Introduction

Every Ruby object carries a hidden, anonymous class called its singleton class (also known as the eigenclass or metaclass). This lesson explains what the singleton class is, how to access it, and why understanding it unlocks the secret behind class methods.

## Key Concepts

- **Singleton Class**: A hidden class unique to a single object that holds methods defined only for that object.
- **Eigenclass**: An older synonym for singleton class, still found in many Ruby books.
- **Class Methods**: Methods defined on a class object, which are technically singleton methods on that class.

## Real World Context

When you write `User.find(1)` in Rails, `find` is not an instance method — it is a singleton method on the `User` class object. Understanding the singleton class explains why class methods are inherited by subclasses and how frameworks like ActiveRecord build their query interfaces.

## Deep Dive

The singleton class holds methods unique to a single object. You can add a method to one specific string without affecting any other string:

```ruby
obj = "hello"

# Add method to this string only
def obj.shout
  upcase + "!!!"
end

obj.shout        # => "HELLO!!!"
"world".shout    # NoMethodError!
```

Only `obj` has the `shout` method. Other strings are unaffected because the method lives in the singleton class of that specific object.

There are two ways to access an object's singleton class:

```ruby
obj = Object.new

# Method 1: singleton_class method
obj.singleton_class

# Method 2: class << syntax
class << obj
  def exclusive_method
    "only for this object"
  end
end
```

Both approaches give you access to the same hidden class. The `class << obj` syntax opens the singleton class body so you can define multiple methods at once.

Class methods are actually singleton methods on the class object. All three of the following definitions are equivalent:

```ruby
class User
  # These are equivalent:
  def self.find(id)
    # ...
  end
end

# Same as:
class << User
  def find(id)
    # ...
  end
end

# Same as:
def User.find(id)
  # ...
end
```

Each version adds the `find` method to `User`'s singleton class. Because `User` is itself an object (an instance of `Class`), it can have singleton methods just like any other object.

Singleton classes also form their own inheritance chain, which is why class methods are inherited:

```ruby
class Animal; end
class Dog < Animal; end

Dog.singleton_class.superclass == Animal.singleton_class  # true!
```

Since `Dog`'s singleton class inherits from `Animal`'s singleton class, any class method defined on `Animal` is also available on `Dog`.

## Common Pitfalls

1. **Confusing singleton methods with instance methods** — Writing `def self.find` inside a class body defines a class method (singleton method on the class), not an instance method. Calling `User.new.find` will raise `NoMethodError`.
2. **Forgetting that Integer (immediate values)/Symbol/true/false/nil cannot have singleton classes** — Immediate values in Ruby share instances, so you cannot define singleton methods on them.

## Best Practices

1. **Use `def self.method_name` for clarity** — Although `class << self` works, `def self.method_name` makes it immediately obvious that you are defining a class method.
2. **Check with `singleton_methods`** — Call `obj.singleton_methods` to list all singleton methods on an object, which is helpful when debugging metaprogrammed code.

## Summary

- Every Ruby object has a hidden singleton class that holds methods unique to that object.
- Class methods are singleton methods on the class object, which is why they are inherited by subclasses.
- You can access the singleton class with `obj.singleton_class` or the `class << obj` syntax.

## Code Examples

**Demonstrating how class methods are inherited through the singleton class chain**

```ruby
class Animal
  def self.kingdom
    "Animalia"
  end
end

class Dog < Animal; end

Dog.kingdom # => "Animalia"
Dog.singleton_class.superclass == Animal.singleton_class # => true
```


## Resources

- [Object#singleton_class](https://docs.ruby-lang.org/en/4.0/Object.html#method-i-singleton_class) — Official Ruby documentation for accessing an object's singleton class.

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*