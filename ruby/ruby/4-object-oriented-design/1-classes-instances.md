---
source_course: "ruby"
source_lesson: "ruby-classes-instances"
---

# Classes and Instances

## Introduction
Classes are the fundamental building blocks of object-oriented Ruby. Every piece of data you work with is an instance of some class, and understanding how to define your own classes is essential for writing organized, reusable code. This lesson covers class definition, constructors, attribute accessors, and the difference between class-level and instance-level state.

## Key Concepts
- **Class**: A blueprint that defines both state (instance variables) and behavior (methods) for its instances.
- **Instance Variable (`@var`)**: A variable that belongs to a specific object instance, prefixed with `@`.
- **Class Variable (`@@var`)**: A variable shared across all instances of a class (and its subclasses), prefixed with `@@`.
- **Attribute Accessor**: Shortcut macros (`attr_reader`, `attr_writer`, `attr_accessor`) that generate getter and/or setter methods automatically.
- **Constructor (`initialize`)**: A special method called automatically when you create a new instance with `.new`.

## Real World Context
In any production Rails application, you define dozens of classes to model your domain: `User`, `Order`, `Payment`. Understanding how to structure classes with proper accessors, constructors, and class-level methods is the foundation of every Ruby project. For example, a background job system might use a class variable to track how many jobs have been enqueued, while each job instance holds its own payload in instance variables.

## Deep Dive
Ruby classes define both state (instance variables) and behavior (methods). The `initialize` method serves as the constructor, called automatically when you invoke `.new`.

Here is a basic class with a constructor and an instance method:

```ruby
class Person
  # Constructor
  def initialize(name, age)
    @name = name   # Instance variable
    @age = age
  end

  # Instance method
  def introduce
    "Hi, I'm #{@name} and I'm #{@age}"
  end
end

person = Person.new("Alice", 30)
person.introduce  # => "Hi, I'm Alice and I'm 30"
```

The `@name` and `@age` variables are private to each `Person` instance. You cannot access them from outside without defining getter methods.

### Attribute Accessors

Ruby provides shortcut macros for generating getter and setter methods, so you don't have to write boilerplate:

```ruby
class Person
  attr_reader :name        # Only getter
  attr_writer :age         # Only setter
  attr_accessor :email     # Both getter and setter

  def initialize(name, age, email)
    @name = name
    @age = age
    @email = email
  end
end

person = Person.new("Bob", 25, "bob@example.com")
person.name              # => "Bob"
person.age = 26          # Sets @age
person.email = "new@ex.com"  # Sets @email
```

`attr_reader` creates a getter, `attr_writer` creates a setter, and `attr_accessor` creates both. These are just class-level method calls that generate methods for you behind the scenes.

### Class Variables vs Instance Variables

Class variables (`@@`) are shared across all instances, while instance variables (`@`) belong to a single object:

```ruby
class Counter
  @@count = 0              # Class variable (shared)

  def initialize
    @id = (@@count += 1)   # Instance variable (per object)
  end

  def self.total           # Class method
    @@count
  end

  attr_reader :id
end

a = Counter.new   # a.id => 1
b = Counter.new   # b.id => 2
Counter.total     # => 2
```

The `self.total` method is a class method, called on `Counter` itself rather than on an instance. The `@@count` variable is incremented every time a new instance is created, demonstrating shared state.

## Common Pitfalls
1. **Using class variables (`@@`) with inheritance** — Class variables are shared across the entire inheritance tree, not just one class. A subclass modifying `@@count` changes it for the parent too. Prefer class-level instance variables (`@count` on `self`) or `class_attribute` in Rails.
2. **Forgetting that instance variables default to `nil`** — If you reference `@name` before assigning it, Ruby returns `nil` silently rather than raising an error. Always initialize all instance variables in `initialize`.

## Best Practices
1. **Use `attr_accessor` sparingly** — Only expose setters when mutation is genuinely needed. Prefer `attr_reader` for read-only attributes to keep objects more predictable.
2. **Initialize all instance variables in the constructor** — This makes the class's state obvious and prevents subtle `nil` bugs when methods are called before variables are set.

## Summary
- Ruby classes bundle state (instance variables) with behavior (methods) and use `initialize` as the constructor.
- `attr_reader`, `attr_writer`, and `attr_accessor` generate getter/setter methods automatically.
- Class variables (`@@`) are shared across all instances and subclasses; prefer class-level instance variables for safer shared state.

## Code Examples

**Defining a class with attribute accessors, a constructor, and an instance method**

```ruby
class Person
  attr_reader :name
  attr_accessor :email

  def initialize(name, email)
    @name = name
    @email = email
  end

  def introduce
    "Hi, I'm #{@name} (#{@email})"
  end
end

person = Person.new("Alice", "alice@example.com")
person.introduce  # => "Hi, I'm Alice (alice@example.com)"
person.email = "new@example.com"
person.name       # => "Alice" (read-only)
```


## Resources

- [Class Documentation](https://docs.ruby-lang.org/en/4.0/Class.html) — Official Class documentation

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*