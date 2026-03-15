---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-eval-family"
---

# Instance Eval & Class Eval

## Introduction

`instance_eval` and `class_eval` let you change what `self` refers to inside a block, giving you access to an object's private internals or letting you define methods on a class from the outside. These two methods are the backbone of most Ruby DSLs.

## Key Concepts

- **instance_eval**: Evaluates a block with `self` set to the receiver object. Defines methods on the receiver's singleton class.
- **class_eval (module_eval)**: Evaluates a block in the context of a class or module. Defines instance methods on that class.
- **DSL (Domain-Specific Language)**: A mini-language built on top of Ruby, often powered by `instance_eval` to provide a clean configuration syntax.

## Real World Context

Gemfiles, RSpec tests, and Rails routes all use `instance_eval` behind the scenes. When you write `gem 'rails'` in a Gemfile, Bundler passes the file contents to `instance_eval` on a `Dsl` object, so bare method calls like `gem` and `source` are dispatched to that object. Understanding these eval methods lets you build — and debug — similar DSLs.

## Deep Dive

`instance_eval` evaluates a block with `self` set to the receiver object. This gives you access to private methods and instance variables:

```ruby
class Person
  def initialize
    @secret = "hidden"
  end
end

person = Person.new

# Access private state
person.instance_eval { @secret }  # => "hidden"

# Add singleton method
person.instance_eval do
  def speak
    "I know #{@secret}"
  end
end
```

Inside the block, `self` is `person`, so `@secret` resolves to that object's instance variable and `def speak` defines a singleton method on `person` alone.

`class_eval` (aliased as `module_eval`) evaluates code in the context of a class or module, making `def` define instance methods:

```ruby
User.class_eval do
  def greet
    "Hello!"
  end
end

User.new.greet  # => "Hello!"
```

The `greet` method is now an instance method on `User`, available to all instances.

The critical difference between the two becomes clear when you use them on a class:

```ruby
class User; end

# instance_eval: defines SINGLETON methods (class methods)
User.instance_eval do
  def find(id); end  # User.find(1) works
end

# class_eval: defines INSTANCE methods
User.class_eval do
  def save; end      # User.new.save works
end
```

`instance_eval` on `User` changes `self` to the `User` object, so `def find` becomes a singleton method (class method). `class_eval` opens the class body, so `def save` becomes an instance method.

A common DSL pattern uses `instance_eval` to create clean configuration blocks:

```ruby
class ConfigBuilder
  def initialize(&block)
    instance_eval(&block) if block
  end

  def port(n)
    @port = n
  end

  def host(h)
    @host = h
  end
end

config = ConfigBuilder.new do
  port 8080
  host "localhost"
end
```

Inside the block, `self` is the `ConfigBuilder` instance, so `port` and `host` are method calls on it rather than local variable assignments.

## Common Pitfalls

1. **Mixing up instance_eval and class_eval on a class** — `User.instance_eval { def x; end }` creates a class method, not an instance method. If you want instance methods, use `class_eval`.
2. **Losing access to outer variables in string eval** — `instance_eval("string")` evaluates a string, which has security risks and does not form a closure. Prefer block form whenever possible.

## Best Practices

1. **Use class_eval for adding instance methods** — It is more readable and clearly signals intent compared to reopening the class with the `class` keyword.
2. **Use instance_eval for DSLs** — When building configuration or builder objects, `instance_eval` provides the cleanest user-facing syntax.

## Summary

- `instance_eval` changes `self` to the receiver, granting access to its private state and defining singleton methods.
- `class_eval` opens a class body, making `def` define instance methods on that class.
- Together they power most Ruby DSLs, from Gemfiles to RSpec.

## Code Examples

**Building a mini router DSL with instance_eval**

```ruby
class Router
  attr_reader :routes

  def initialize(&block)
    @routes = []
    instance_eval(&block) if block
  end

  def get(path, &handler)
    @routes << { method: :get, path: path, handler: handler }
  end
end

router = Router.new do
  get '/hello' do
    'Hello, world!'
  end
end

router.routes.first[:path] # => "/hello"
```


## Resources

- [BasicObject#instance_eval](https://docs.ruby-lang.org/en/4.0/BasicObject.html#method-i-instance_eval) — Official Ruby documentation for instance_eval, covering block and string forms.

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*