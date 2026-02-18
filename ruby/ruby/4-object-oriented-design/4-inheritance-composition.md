---
source_course: "ruby"
source_lesson: "ruby-inheritance-composition"
---

# Single Inheritance

Ruby supports single inheritance:

```ruby
class Animal
  def breathe
    "breathing"
  end
end

class Dog < Animal
  def bark
    "woof!"
  end
end

dog = Dog.new
dog.breathe  # Inherited
dog.bark     # Own method
```

## Super Keyword

```ruby
class Animal
  def speak
    "..."
  end
end

class Dog < Animal
  def speak
    super + " Woof!"  # Calls parent's speak
  end
end

Dog.new.speak  # => "... Woof!"
```

## Composition Over Inheritance

Prefer composition when inheritance doesn't represent an "is-a" relationship:

```ruby
# Instead of:
class Car < Engine  # Wrong! Car is not an Engine

# Use composition:
class Car
  def initialize
    @engine = Engine.new  # Car has an Engine
  end

  def start
    @engine.ignite
  end
end
```

## Duck Typing

Ruby doesn't require formal interfaces. If it responds to the right methods, it works:

```ruby
def process(printer)
  printer.print("Hello")  # Works with any object that has #print
end

# Any of these work:
process(ConsolePrinter.new)
process(FilePrinter.new)
process(NetworkPrinter.new)
```

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*