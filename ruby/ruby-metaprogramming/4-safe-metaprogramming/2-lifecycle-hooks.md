---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-lifecycle-hooks"
---

# Lifecycle Hooks

## Introduction

Ruby calls special callback methods when classes and modules change at runtime. These hooks let you react to events like module inclusion, class inheritance, and method definition. Understanding lifecycle hooks is essential for building frameworks and plugins that configure themselves automatically.

## Key Concepts

- **`included` hook**: Called on a module when it is included into a class, receiving the including class as an argument
- **`inherited` hook**: Called on a class when a subclass is created, receiving the new subclass as an argument
- **`method_added` hook**: Called on a class whenever a new instance method is defined on it
- **`const_missing` hook**: Called when a reference to an undefined constant is encountered

## Real World Context

Rails uses `included` extensively. When you write `include ActiveModel::Validations` in a model, the `included` hook automatically extends your class with class-level methods like `validates` and sets up internal data structures. Without this hook, every developer would have to manually call multiple setup methods after each `include`.

## Deep Dive

### included / extended

The `included` hook fires when a module is mixed into a class. This is the most commonly used hook in Ruby metaprogramming. The following example shows how to use it to automatically extend a class with additional class methods:

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

The `included` callback receives `base` (the class doing the including), which lets you call `extend`, `class_eval`, or any other class-level configuration.

### inherited

The `inherited` hook lets a parent class react whenever a new subclass is defined. This is useful for registering subclasses in a plugin system:

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

Note that `GrandChild` triggers `inherited` on `Child`, not on `Base`, because each class only sees its direct subclasses.

### method_added / method_removed

These hooks fire whenever an instance method is defined or removed on a class. Here is a simple example:

```ruby
class Observed
  def self.method_added(name)
    puts "Method added: #{name}"
  end

  def foo; end  # "Method added: foo"
end
```

This is useful for auto-registration or validation of method signatures at definition time.

### const_missing

When Ruby encounters an undefined constant, it calls `const_missing` before raising a `NameError`. You can override this to implement autoloading:

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

The hook receives the constant name as a symbol, giving you the opportunity to dynamically define or load it.

## Common Pitfalls

1. **Infinite recursion in `method_added`** — If your `method_added` hook defines a new method, it triggers itself again. Guard against this with a flag variable or by checking the method name.
2. **Forgetting that `inherited` only fires for direct subclasses** — `Base.inherited` is not called when `GrandChild` subclasses `Child`. Each class only sees its own direct children.

## Best Practices

1. **Use `included` with `base.extend(ClassMethods)`** — This is the standard Ruby pattern for adding both instance and class methods from a module. It is so common that ActiveSupport::Concern wraps it.
2. **Keep hooks lightweight** — Hooks run synchronously at class definition time. Heavy work (file I/O, network calls) in hooks slows down application boot.

## Summary

- Ruby provides hooks like `included`, `inherited`, `method_added`, and `const_missing` that fire during class/module lifecycle events
- These hooks receive the relevant object (class, subclass, method name) as an argument, enabling automatic configuration
- They are the foundation for framework patterns like Rails' `ActiveSupport::Concern`

## Code Examples

**Using the included hook to build a self-registering plugin system**

```ruby
module PluginRegistry
  def self.included(base)
    base.instance_variable_set(:@plugins, [])
    base.extend(ClassMethods)
  end

  module ClassMethods
    def register_plugin(name)
      @plugins << name
    end

    def plugins
      @plugins
    end
  end
end

class App
  include PluginRegistry
  register_plugin :auth
  register_plugin :logging
end

App.plugins  # => [:auth, :logging]
```


## Resources

- [Module Hooks](https://docs.ruby-lang.org/en/4.0/Module.html) — Official Module documentation covering included, extended, and other lifecycle hooks

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*