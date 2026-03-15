---
source_course: "ruby"
source_lesson: "ruby-access-control"
---

# Access Control (Private/Protected)

## Introduction
Encapsulation is a cornerstone of good object-oriented design, and Ruby provides three visibility levels — public, private, and protected — to control which methods are part of your class's external API. Understanding when to use each level helps you write classes with clean interfaces and hidden implementation details. This lesson covers all three visibility keywords plus the inline declaration syntax.

## Key Concepts
- **Public**: The default visibility. Public methods can be called by anyone from anywhere.
- **Private**: Methods that can only be called within the defining class without an explicit receiver (Ruby 2.7+ allows `self.private_method`).
- **Protected**: Methods that can be called by any instance of the same class or its subclasses, but not from outside.
- **Inline Declaration**: Prefixing a `def` with `private` or `protected` to declare visibility on a single method.

## Real World Context
In a payment processing system, the `charge` method might be public, but the internal `calculate_tax` and `apply_discount` helpers should be private — they are implementation details that callers should never depend on. Protected methods are rarer but useful when instances of the same class need to compare internal state, such as comparing ages or salaries between two `Employee` objects.

## Deep Dive
Ruby provides three levels of method visibility to control encapsulation.

### Public (Default)

All methods are public by default. They can be called by anyone with a reference to the object:

```ruby
class Person
  def greet
    "Hello!"
  end
end
```

The `greet` method is accessible from anywhere because no visibility modifier is applied.

### Private

Private methods can only be called on `self` — you cannot call them with an explicit receiver (though Ruby 2.7+ relaxed this to allow `self.private_method`):

```ruby
class Person
  def public_method
    private_helper  # OK
    self.other_private  # OK in Ruby 2.7+
  end

  private

  def private_helper
    "secret"
  end

  def other_private
    "also secret"
  end
end

person = Person.new
person.private_helper  # NoMethodError!
```

Everything defined after the `private` keyword becomes private. External callers get a `NoMethodError` if they try to invoke these methods directly.

### Protected

Protected methods can be called by any instance of the same class or its subclasses. This is useful when objects need to compare internal state:

```ruby
class Person
  def compare_age(other)
    self.age <=> other.age  # Can access other's protected method
  end

  protected

  def age
    @age
  end
end
```

The key difference from private is that `other.age` works here because both objects are instances of `Person`. With private, calling `other.private_method` would raise an error.

### Inline Declaration

You can also declare visibility inline for individual methods, which keeps the visibility right next to the method definition:

```ruby
class Example
  private def helper
    "inline private"
  end

  protected def shared
    "inline protected"
  end
end
```

This style avoids the ambiguity of "how far does the `private` keyword reach" when reading a long class file.

## Common Pitfalls
1. **Assuming private means truly inaccessible** — In Ruby, `send(:private_method)` bypasses visibility checks entirely. Private is a signal to other developers, not an enforced security boundary.
2. **Overusing protected** — Protected is rarely needed. Most of the time, you want either public (part of the API) or private (implementation detail). Protected is only for inter-instance collaboration within the same class hierarchy.

## Best Practices
1. **Start private, promote as needed** — Make helper methods private by default. Only make them public when external callers genuinely need them. This keeps your class's public API small and stable.
2. **Use inline declaration for clarity in large classes** — In classes with many methods, the inline `private def` style makes it immediately clear which methods are private without scrolling to find a `private` keyword block.

## Summary
- Public is the default; private restricts to self-calls only; protected allows same-class instance access.
- Ruby 2.7+ allows `self.private_method` calls, relaxing the old strict rule.
- Prefer starting methods as private and promoting to public only when needed to keep APIs clean.

## Code Examples

**A class demonstrating public, protected, and private method visibility**

```ruby
class BankAccount
  attr_reader :owner

  def initialize(owner, balance)
    @owner = owner
    @balance = balance
  end

  def deposit(amount)
    @balance += validate_amount(amount)
  end

  def >(other)
    balance > other.balance
  end

  protected

  def balance
    @balance
  end

  private

  def validate_amount(amount)
    raise ArgumentError, "Must be positive" unless amount.positive?
    amount
  end
end
```


## Resources

- [Modules and Classes Syntax](https://docs.ruby-lang.org/en/4.0/syntax/modules_and_classes_rdoc.html) — Official Ruby documentation covering access control in modules and classes

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*