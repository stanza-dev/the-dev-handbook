---
source_course: "ruby"
source_lesson: "ruby-access-control"
---

# Visibility Keywords

Ruby provides three levels of method visibility:

## Public (Default)

Can be called by anyone:

```ruby
class Person
  def greet
    "Hello!"
  end
end
```

## Private

Can only be called on `self` (no explicit receiver, though Ruby 2.7+ allows `self.private_method`):

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

## Protected

Can be called by any instance of the *same class* or subclass:

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

## Inline Declaration

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

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*