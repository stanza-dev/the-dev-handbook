---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-singleton-class"
---

# Every Object Has a Hidden Class

The singleton class (also called eigenclass or metaclass) holds methods unique to a single object.

```ruby
obj = "hello"

# Add method to this string only
def obj.shout
  upcase + "!!!"
end

obj.shout        # => "HELLO!!!"
"world".shout    # NoMethodError!
```

## Accessing the Singleton Class

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

## Class Methods ARE Singleton Methods

Class methods are actually singleton methods on the class object:

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

## The Singleton Class Chain

```ruby
class Animal; end
class Dog < Animal; end

Dog.singleton_class.superclass == Animal.singleton_class  # true!
```

This is why class methods are inherited!

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*