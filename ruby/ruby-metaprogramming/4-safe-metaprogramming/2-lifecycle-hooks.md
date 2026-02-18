---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-lifecycle-hooks"
---

# Class & Module Hooks

Ruby calls special methods when certain events occur.

## included / extended

```ruby
module Trackable
  def self.included(base)
    puts "#{self} was included in #{base}"
    base.extend(ClassMethods)
  end

  def self.extended(base)
    puts "#{self} was extended by #{base}"
  end

  module ClassMethods
    def track
      puts "Tracking #{self}"
    end
  end
end

class User
  include Trackable  # "Trackable was included in User"
end

User.track  # "Tracking User"
```

## inherited

```ruby
class Base
  def self.inherited(subclass)
    puts "#{subclass} inherits from #{self}"
    @subclasses ||= []
    @subclasses << subclass
  end

  def self.subclasses
    @subclasses || []
  end
end

class Child < Base; end   # "Child inherits from Base"
class GrandChild < Child; end  # "GrandChild inherits from Child"

Base.subclasses  # => [Child]
```

## method_added / method_removed

```ruby
class Observed
  def self.method_added(name)
    puts "Method added: #{name}"
  end

  def foo; end  # "Method added: foo"
end
```

## const_missing

```ruby
class AutoLoader
  def self.const_missing(name)
    puts "Trying to load: #{name}"
    # Could load from file, define class, etc.
    const_set(name, Class.new)
  end
end

AutoLoader::SomeClass  # "Trying to load: SomeClass"
```

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*