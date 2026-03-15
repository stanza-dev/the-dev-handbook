---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-method-objects"
---

# Method Objects

## Introduction

In Ruby, methods are not just syntax — they can be captured as first-class objects, stored in variables, passed to other methods, and introspected for metadata like arity and source location. This lesson explores the `Method` and `UnboundMethod` classes.

## Key Concepts

- **Method object**: A bound reference to a method on a specific receiver, obtained with `obj.method(:name)`.
- **UnboundMethod**: A method reference without a receiver, obtained with `Class.instance_method(:name)`. Must be bound before calling.
- **arity**: The number of required arguments a method accepts.

## Real World Context

Debugging tools like Pry use `Method#source_location` to show you exactly where a method is defined. When you type `show-source User#save` in Pry, it grabs the `Method` object, reads its source location, and opens the file. Understanding method objects lets you build similar introspection tools.

## Deep Dive

Ruby lets you capture methods as objects and pass them around:

```ruby
class Calculator
  def add(a, b)
    a + b
  end
end

calc = Calculator.new
m = calc.method(:add)  # Get Method object

m.class        # => Method
m.call(2, 3)   # => 5
m.arity        # => 2 (number of required arguments)
m.owner        # => Calculator
m.receiver     # => calc
```

The `Method` object `m` is bound to `calc`, meaning you can call it later without referencing `calc` again. The `owner` tells you which class defined the method, and `receiver` tells you which object it is bound to.

You can also get an unbound method that is not tied to any specific instance:

```ruby
class Calculator
  def add(a, b); a + b; end
end

# Get unbound method (no receiver)
um = Calculator.instance_method(:add)
um.class  # => UnboundMethod

# Bind to an instance to call it
calc = Calculator.new
bound = um.bind(calc)
bound.call(2, 3)  # => 5
```

An `UnboundMethod` cannot be called directly — you must bind it to an instance of the class (or a subclass) first. This is useful for transplanting methods between objects.

Method objects have several practical applications:

```ruby
# Pass method as block
def double(n); n * 2; end
[1, 2, 3].map(&method(:double))  # => [2, 4, 6]

# Store methods for later
actions = {
  add: method(:add),
  sub: method(:sub)
}
actions[:add].call(1, 2)

# Find where a method comes from
"hello".method(:upcase).source_location
# => nil (C method) or ["file.rb", 42, 2, 44, 5] in Ruby 4.0
```

The `&method(:double)` pattern converts a method into a block, making it composable with iterators like `map`. `source_location` returns `nil` for C-implemented methods and a five-element array `[file, start_line, start_col, end_line, end_col]` for Ruby-defined methods (Ruby 4.0+; previously two elements).

You can also introspect a method's parameters:

```ruby
m = "hello".method(:gsub)

m.parameters
# => [[:req, :pattern], [:opt, :replacement]]

m.super_method  # => Method for superclass version (if any)
```

The `parameters` method returns an array describing each parameter's type (`:req`, `:opt`, `:rest`, `:keyreq`, `:key`, `:keyrest`, `:block`) and name.

## Common Pitfalls

1. **Calling an UnboundMethod directly** — `UnboundMethod#call` does not exist. You must bind it to an instance first with `.bind(obj)`, or you will get a `TypeError`.
2. **Assuming source_location always works** — Methods implemented in C (like `String#upcase`) return `nil` for `source_location`. Check for `nil` before using the result.

## Best Practices

1. **Use `&method(:name)` for clean functional style** — Instead of `[1,2,3].map { |n| double(n) }`, use `[1,2,3].map(&method(:double))` for concise, point-free code.
2. **Use `source_location` for debugging** — When you cannot figure out where a method comes from (especially in metaprogrammed code), `obj.method(:name).source_location` is your best friend.

## Summary

- `obj.method(:name)` returns a bound `Method` object you can call, store, and introspect.
- `Class.instance_method(:name)` returns an `UnboundMethod` that must be bound before use.
- Method objects expose metadata like `arity`, `parameters`, `owner`, and `source_location` (which returns 5 elements in Ruby 4.0: file, start_line, start_col, end_line, end_col).

## Code Examples

**Capturing a method as an object and using it as a block with map**

```ruby
class Greeter
  def hello(name)
    "Hello, #{name}!"
  end
end

m = Greeter.new.method(:hello)
m.call("Ruby")          # => "Hello, Ruby!"
m.arity                 # => 1
m.owner                 # => Greeter
m.parameters            # => [[:req, :name]]
["Alice", "Bob"].map(&m) # => ["Hello, Alice!", "Hello, Bob!"]
```


## Resources

- [Method Class](https://docs.ruby-lang.org/en/4.0/Method.html) — Official Ruby documentation for the Method class, covering call, arity, source_location, and more.

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*