---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-context-probe"
---

# Context Probing Techniques

## Introduction

Ruby provides a rich set of introspection methods that let you examine any object's state at runtime — its variables, methods, constants, and even what `self` is at any given point. This lesson surveys the most useful context-probing techniques.

## Key Concepts

- **Introspection**: Examining an object's internal state (variables, methods, class) at runtime.
- **instance_variables**: Returns an array of an object's instance variable names.
- **local_variables**: Returns an array of local variable names in the current scope.

## Real World Context

When debugging a complex Rails controller, you might need to know which instance variables are set before a view renders, or which methods are available on a service object. Tools like `binding.irb` and Pry use these introspection methods to provide their interactive debugging experience. Mastering them lets you build your own debugging helpers.

## Deep Dive

You can check what `self` is in any context by printing it:

```ruby
class Demo
  puts self        # => Demo

  def instance_method
    puts self      # => #<Demo:0x...>
  end

  class << self
    puts self      # => #<Class:Demo>
  end
end
```

In the class body, `self` is the class. In an instance method, `self` is the instance. Inside `class << self`, `self` is the singleton class.

You can list all local variables in the current scope:

```ruby
a = 1
b = 2
local_variables  # => [:a, :b]

binding.local_variable_get(:a)  # => 1
```

The `local_variables` method returns symbols for each local variable, and you can read their values through a binding.

Instance variables can be listed and accessed dynamically:

```ruby
@x = 10
@y = 20
instance_variables  # => [:@x, :@y]

instance_variable_get(:@x)  # => 10
instance_variable_set(:@z, 30)
```

These methods are useful for serialization, debugging, and building generic object inspectors.

You can query which methods a class defines and separate public from private:

```ruby
class Example
  def public_method; end
  private
  def private_method; end
end

Example.instance_methods(false)         # => [:public_method]
Example.private_instance_methods(false) # => [:private_method]

# On an instance
obj = Example.new
obj.methods - Object.methods  # Methods unique to Example
```

Passing `false` to `instance_methods` excludes inherited methods, showing only those defined directly on the class. Subtracting `Object.methods` is another way to filter out the noise.

Constants — including nested classes — can be listed and retrieved dynamically:

```ruby
module Container
  CONST_A = 1
  class Inner; end
end

Container.constants  # => [:CONST_A, :Inner]
Container.const_get(:CONST_A)  # => 1
```

Note that classes defined inside a module appear as constants, which is how Ruby's autoloading mechanisms discover them.

## Common Pitfalls

1. **Relying on introspection in production code** — Introspection methods are great for debugging and metaprogramming but can make code harder to understand. Prefer explicit interfaces over runtime discovery in application logic.
2. **Forgetting the `false` argument** — `instance_methods` without `false` returns all inherited methods too, which can be a very long list. Always pass `false` when you want only directly defined methods.

## Best Practices

1. **Use `respond_to?` over `methods.include?`** — To check if an object supports a method, `obj.respond_to?(:name)` is idiomatic and also respects `respond_to_missing?`.
2. **Build debugging helpers with these tools** — Create a `debug_inspect(obj)` method that prints instance variables, singleton methods, and the ancestor chain in one place.

## Summary

- Ruby provides `instance_variables`, `local_variables`, `methods`, and `constants` for runtime introspection.
- Pass `false` to `instance_methods` to see only directly defined methods.
- These tools power debugging consoles, serializers, and metaprogramming frameworks.

## Code Examples

**Iterating over an object's instance variables for debugging or serialization**

```ruby
class User
  attr_accessor :name, :email

  def initialize(name, email)
    @name = name
    @email = email
  end
end

user = User.new("Alice", "alice@example.com")
user.instance_variables.each do |var|
  puts "#{var} = #{user.instance_variable_get(var)}"
end
# @name = Alice
# @email = alice@example.com
```


## Resources

- [BasicObject#instance_eval](https://docs.ruby-lang.org/en/4.0/BasicObject.html#method-i-instance_eval) — Official Ruby documentation for instance_eval, often used alongside context probing techniques.

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*