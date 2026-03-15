---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-method-introspection"
---

# Method Introspection

## Introduction

Ruby provides a rich reflection API for examining methods at runtime. You can discover where a method is defined, what parameters it expects, and which class or module owns it. This capability is invaluable for building developer tools, generating documentation, and debugging complex inheritance hierarchies.

## Key Concepts

- **`Method` object**: A bound reference to a method on a specific receiver, obtained via `obj.method(:name)`
- **`UnboundMethod`**: A method reference not tied to a receiver, obtained via `Class.instance_method(:name)`
- **`source_location`**: Returns `[file, start_line, start_col, end_line, end_col]` in Ruby 4.0 (previously `[file, line]`), or `nil` for C-implemented methods
- **`parameters`**: Returns an array describing each parameter's type (`:req`, `:opt`, `:rest`, `:keyreq`, `:key`, `:keyrest`, `:block`)

## Real World Context

When investigating a bug in a large Rails application, you might wonder which module defines the `authenticate` method that is being called. Running `UserController.instance_method(:authenticate).owner` immediately tells you whether it comes from Devise, your custom concern, or the base controller. This is far faster than searching through dozens of included modules manually.

## Deep Dive

### Method Location

The `source_location` method reveals exactly where a method is defined. This is often the first tool you reach for when debugging:

```ruby
class User
  def greet
    "Hello"
  end
end

m = User.new.method(:greet)
m.source_location  # => ["user.rb", 2, 2, 4, 5] (Ruby 4.0: file, start_line, start_col, end_line, end_col)
m.owner            # => User
m.receiver         # => #<User:...>

# C methods return nil for source_location
"hello".method(:upcase).source_location  # => nil
```

The `owner` tells you which class or module the method belongs to, while `receiver` tells you the specific object it is bound to.

### Method Parameters

The `parameters` method provides a structured description of a method's signature. Each parameter is represented as a `[type, name]` pair:

```ruby
def example(required, optional = 1, *rest, keyword:, default_kw: 2, **kwrest, &block)
end

method(:example).parameters
# => [[:req, :required], [:opt, :optional], [:rest, :rest],
#     [:keyreq, :keyword], [:key, :default_kw], [:keyrest, :kwrest],
#     [:block, :block]]
```

This is useful for building validation logic, auto-generating forms, or introspecting DSL methods.

### Finding All Methods

Ruby provides several methods for listing available methods on a class. The `false` argument controls whether inherited methods are included:

```ruby
class Child < Parent
  include SomeModule
end

# Instance methods defined directly on Child
Child.instance_methods(false)

# All instance methods including inherited
Child.instance_methods

# Methods unique to this class (not on Object)
Child.instance_methods - Object.instance_methods

# Private methods
Child.private_instance_methods(false)
```

Passing `false` to `instance_methods` is particularly useful for understanding what a specific class contributes to the ancestor chain.

### Method Defined Checks

You can check for method existence and retrieve unbound method objects. These are commonly used in metaprogramming to conditionally define or wrap methods:

```ruby
User.method_defined?(:greet)          # => true
User.method_defined?(:greet, false)   # Only if defined directly
User.instance_method(:greet)          # Get UnboundMethod
```

The second argument to `method_defined?` (Ruby 2.6+) lets you check only the class itself, excluding ancestors.

## Common Pitfalls

1. **Expecting `source_location` for C methods** — Built-in methods implemented in C return `nil` from `source_location`. Do not assume every method has a Ruby source file.
2. **Confusing `methods` with `instance_methods`** — `obj.methods` returns methods callable on the object; `Class.instance_methods` returns methods defined on the class for its instances. Mixing them up leads to unexpected results.

## Best Practices

1. **Use `owner` to trace method origin** — When debugging, `method(:foo).owner` is the fastest way to find which module or class contributes a specific method.
2. **Pass `false` to `instance_methods` for focused results** — This filters out inherited methods, making it easy to see what a single class defines.

## Summary

- `Method` and `UnboundMethod` objects provide `source_location`, `owner`, `parameters`, and more
- `instance_methods(false)` reveals only the methods defined directly on a class, excluding ancestors
- These introspection tools are essential for debugging, tooling, and metaprogramming

## Code Examples

**A utility that maps every method on an object to its owner and source location**

```ruby
# Find where all methods on an object are defined
def method_map(obj)
  obj.methods.sort.each_with_object({}) do |name, map|
    m = obj.method(name)
    loc = m.source_location  # Ruby 4.0: [file, start_line, start_col, end_line, end_col]
    map[name] = {
      owner: m.owner,
      location: loc ? "#{loc[0]}:#{loc[1]}:#{loc[2]}" : "(native)"
    }
  end
end

method_map("hello").select { |_, v| v[:owner] == String }.first(3)
```


## Resources

- [Object#methods](https://docs.ruby-lang.org/en/4.0/Object.html#method-i-methods) — Official documentation for Object method introspection including methods, respond_to?, and method objects

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*