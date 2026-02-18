---
source_course: "ruby"
source_lesson: "ruby-classes-instances"
---

# Defining Classes

Ruby classes define both state (instance variables) and behavior (methods):

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

## Attribute Accessors

Ruby provides shortcuts for getter/setter methods:

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

## Class vs Instance

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

## Resources

- [Class Documentation](https://docs.ruby-lang.org/en/4.0/Class.html) — Official Class documentation

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*