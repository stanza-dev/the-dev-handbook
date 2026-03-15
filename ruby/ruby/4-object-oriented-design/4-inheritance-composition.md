---
source_course: "ruby"
source_lesson: "ruby-inheritance-composition"
---

# Inheritance vs Composition

## Introduction
Choosing between inheritance and composition is one of the most important design decisions in object-oriented programming. Ruby supports single inheritance with a clear `<` syntax and encourages composition through duck typing and module mixins. This lesson covers when to inherit, when to compose, and how Ruby's duck typing philosophy influences your design choices.

## Key Concepts
- **Single Inheritance**: Ruby classes can inherit from exactly one parent class using `<`. The child gains all parent methods.
- **`super` Keyword**: Calls the same-named method from the parent class, allowing the child to extend rather than replace behavior.
- **Composition**: Building objects by combining smaller, focused collaborators rather than inheriting from a monolithic parent.
- **Duck Typing**: Ruby does not require formal interfaces. If an object responds to the expected methods, it works — "if it quacks like a duck, it's a duck."

## Real World Context
Consider a notification system that sends emails, SMS, and push notifications. You might be tempted to create a `Notification` base class and inherit `EmailNotification`, `SmsNotification`, etc. But each delivery mechanism is fundamentally different. Composition works better: a `Notification` object has a `delivery_strategy` that handles the transport. Duck typing means any object with a `deliver` method works, making the system easy to extend without modifying existing code.

## Deep Dive
Ruby supports single inheritance — each class can have at most one direct parent.

### Single Inheritance

The `<` operator establishes an inheritance relationship:

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

`Dog` inherits all of `Animal`'s methods. Ruby searches the ancestor chain — `Dog`, then `Animal`, then `Object`, then `Kernel`, then `BasicObject` — when resolving method calls.

### The super Keyword

`super` calls the parent's implementation of the same method, letting you extend behavior:

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

Calling `super` without parentheses forwards all arguments automatically. Use `super()` to call the parent with no arguments explicitly.

### Composition Over Inheritance

Prefer composition when the relationship is "has-a" rather than "is-a":

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

The `Car` delegates to its `Engine` collaborator. This is more flexible because you can swap engines, test with mock engines, or change the engine implementation without affecting the `Car` interface.

### Duck Typing

Ruby doesn't require formal interfaces. If an object responds to the right methods, it works:

```ruby
def process(printer)
  printer.print("Hello")  # Works with any object that has #print
end

# Any of these work:
process(ConsolePrinter.new)
process(FilePrinter.new)
process(NetworkPrinter.new)
```

All three printer objects work because they respond to `print`. No shared base class or interface declaration is needed. This is the heart of Ruby's flexible design philosophy.

## Common Pitfalls
1. **Inheriting for code reuse alone** — Inheritance should model "is-a" relationships. If you inherit just to get some methods, use a module mixin or composition instead. Deep inheritance trees become brittle and hard to understand.
2. **Calling `super` without understanding argument forwarding** — `super` (no parens) forwards all arguments; `super()` forwards none. Mixing these up causes unexpected `ArgumentError` exceptions.

## Best Practices
1. **Follow the "is-a" vs "has-a" test** — Ask: "Is a Car an Engine?" No, so use composition. "Is a Dog an Animal?" Yes, so inheritance is appropriate. This simple test prevents most inheritance misuse.
2. **Leverage duck typing instead of type checking** — Avoid `is_a?` checks in method bodies. Instead, trust that the caller passes an object with the right interface. This makes your code more flexible and easier to test with mocks.

## Summary
- Ruby supports single inheritance with `<`; use `super` to extend parent behavior.
- Prefer composition ("has-a") over inheritance ("is-a") when the relationship is about collaboration, not identity.
- Duck typing means any object with the right methods will work — no formal interfaces needed.

## Code Examples

**Composition with duck typing: swapping collaborators without changing the Car class**

```ruby
# Composition over inheritance
class Engine
  def ignite = puts("Engine started")
end

class Car
  def initialize(engine: Engine.new)
    @engine = engine
  end

  def start
    @engine.ignite
  end
end

# Duck typing: any object with #ignite works
class ElectricMotor
  def ignite = puts("Motor humming")
end

Car.new(engine: ElectricMotor.new).start
# => "Motor humming"
```


## Resources

- [Module Documentation](https://docs.ruby-lang.org/en/4.0/Module.html) — Official Module documentation covering mixins as an alternative to inheritance

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*