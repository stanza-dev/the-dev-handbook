---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-method-missing"
---

# Ghost Methods with method_missing

## Introduction

When Ruby cannot find a method anywhere in the ancestor chain, it calls `method_missing` as a last resort. By overriding this hook, you can intercept calls to methods that do not exist and handle them dynamically. These phantom methods are often called ghost methods.

## Key Concepts

- **method_missing**: A callback Ruby invokes when no method matching the call is found in the ancestor chain.
- **Ghost Method**: A method that does not exist in any class or module but is handled dynamically via `method_missing`.
- **respond_to_missing?**: The companion method you must override alongside `method_missing` to keep introspection consistent.

## Real World Context

Early versions of ActiveRecord used `method_missing` to implement dynamic finders like `User.find_by_name("Alice")`. When you called a method starting with `find_by_`, Rails intercepted it, parsed the attribute name, and built a database query on the fly. This pattern made the API feel magical but came with performance and debugging trade-offs.

## Deep Dive

When Ruby cannot find a method, it calls `method_missing` as a last resort:

```ruby
class Ghost
  def method_missing(name, *args)
    puts "You called #{name} with #{args}"
  end
end

g = Ghost.new
g.anything(1, 2, 3)  # "You called anything with [1, 2, 3]"
```

The `name` parameter is a symbol representing the method that was called, and `args` contains the arguments. Because `anything` is not defined anywhere, Ruby falls through the entire ancestor chain and lands in `method_missing`.

Here is a practical example that implements dynamic finders:

```ruby
class User
  def self.method_missing(name, *args)
    if name.to_s.start_with?("find_by_")
      attribute = name.to_s.sub("find_by_", "")
      find_by_attribute(attribute, args.first)
    else
      super  # Important: call super if not handled!
    end
  end

  private

  def self.find_by_attribute(attr, value)
    # Database query...
    "Finding by #{attr} = #{value}"
  end
end

User.find_by_name("Alice")   # Works!
User.find_by_email("a@b.c")  # Works!
```

The `else super` clause is critical. Without it, genuinely missing methods would be silently swallowed instead of raising `NoMethodError`.

You must always implement `respond_to_missing?` alongside `method_missing` to keep the object's introspection consistent:

```ruby
class Ghost
  def method_missing(name, *args)
    # ...
  end

  def respond_to_missing?(name, include_private = false)
    name.to_s.start_with?("find_by_") || super
  end
end

# Without respond_to_missing?:
g.respond_to?(:find_by_x)  # => false (wrong!)

# With respond_to_missing?:
g.respond_to?(:find_by_x)  # => true (correct!)
```

Without `respond_to_missing?`, `respond_to?` returns false for ghost methods and `method(:find_by_x)` raises `NameError`, breaking duck typing and debugging tools.

## Common Pitfalls

1. **Forgetting `respond_to_missing?`** — This is the most common mistake. Without it, your object lies about its capabilities, breaking code that checks `respond_to?` before calling a method.
2. **Not calling `super` for unhandled methods** — If your `method_missing` does not call `super` when it cannot handle a method, real errors are silently swallowed and bugs become very hard to trace.

## Best Practices

1. **Prefer `define_method` over `method_missing`** — If you know the set of methods at class-load time, generate them with `define_method`. Ghost methods should be a last resort for truly open-ended APIs.
2. **Always benchmark** — `method_missing` is slower than a regular method call because Ruby must walk the entire ancestor chain first. Measure the impact in hot paths.

## Summary

- `method_missing` intercepts calls to undefined methods and lets you handle them dynamically.
- Always implement `respond_to_missing?` alongside it to keep introspection accurate.
- Call `super` for methods you do not handle, and prefer `define_method` when the method set is known in advance.

## Code Examples

**A hash-backed struct using method_missing with proper respond_to_missing?**

```ruby
class FlexibleStruct
  def initialize(hash)
    @data = hash
  end

  def method_missing(name, *args)
    key = name.to_s.chomp('=').to_sym
    if name.to_s.end_with?('=')
      @data[key] = args.first
    elsif @data.key?(key)
      @data[key]
    else
      super
    end
  end

  def respond_to_missing?(name, include_private = false)
    @data.key?(name.to_s.chomp('=').to_sym) || super
  end
end

fs = FlexibleStruct.new(name: "Alice")
fs.name       # => "Alice"
fs.name = "Bob"
fs.name       # => "Bob"
```


## Resources

- [BasicObject#method_missing](https://docs.ruby-lang.org/en/4.0/BasicObject.html#method-i-method_missing) — Official Ruby documentation for method_missing, the hook called when no method is found.

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*