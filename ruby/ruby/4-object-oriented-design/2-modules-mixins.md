---
source_course: "ruby"
source_lesson: "ruby-modules-mixins"
---

# Modules

Ruby does not support multiple inheritance. Instead, it uses Modules for code reuse.

## Include: Instance Methods

Adds module methods as **instance methods**:

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

## Extend: Class Methods

Adds module methods as **class methods**:

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

## Prepend: Method Interception

Inserts the module *before* the class in the ancestor chain:

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

## Resources

- [Module Class](https://docs.ruby-lang.org/en/4.0/Module.html) — Official Module documentation

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*