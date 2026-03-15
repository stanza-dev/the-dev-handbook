---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-define-method"
---

# Defining Methods Dynamically

## Introduction

`define_method` lets you create methods at runtime with computed names and closure-captured variables. This lesson explains how it works, how it differs from `def`, and when to reach for it.

## Key Concepts

- **define_method**: A `Module` method that creates an instance method from a block, with a name that can be computed at runtime.
- **Closure**: A block that captures and retains access to local variables from the scope where it was created.
- **Scope gate**: A keyword (`class`, `module`, `def`) that starts a fresh local-variable scope, blocking access to outer variables.

## Real World Context

ActiveRecord uses `define_method` behind the scenes to generate getter and setter methods for every column in a database table. Instead of writing hundreds of methods by hand, it reads the schema and defines them in a loop. Any time you have a family of methods that follow the same pattern, `define_method` can eliminate the duplication.

## Deep Dive

You create methods at runtime using `define_method`, passing a name and a block that becomes the method body:

```ruby
class User
  ATTRIBUTES = [:name, :email, :age]

  ATTRIBUTES.each do |attr|
    define_method(attr) do
      instance_variable_get("@#{attr}")
    end

    define_method("#{attr}=") do |value|
      instance_variable_set("@#{attr}", value)
    end
  end
end

user = User.new
user.name = "Alice"  # Uses dynamically defined setter
user.name            # Uses dynamically defined getter
```

This loop generates six methods (a getter and setter for each attribute) from a single template. The `attr` variable is captured by the block closure, so each method remembers which attribute it handles.

Blocks passed to `define_method` are closures, meaning they capture variables from the surrounding scope:

```ruby
class Multiplier
  [2, 3, 5, 10].each do |factor|
    define_method("times_#{factor}") do |n|
      n * factor  # factor is captured from the block
    end
  end
end

m = Multiplier.new
m.times_2(5)   # => 10
m.times_10(5)  # => 50
```

Each iteration captures a different value of `factor`, so `times_2` always multiplies by 2 and `times_10` always multiplies by 10.

The key difference between `def` and `define_method` is scope. `def` creates a scope gate that blocks access to outer variables, while `define_method` uses a closure that preserves them:

| Feature | def | define_method |
|---------|-----|---------------|
| Speed | Faster | Slightly slower |
| Scope | New scope (can't access outer vars) | Closure (captures outer vars) |
| Dynamic name | No | Yes |

The following example shows why `define_method` is necessary when the method name is computed:

```ruby
prefix = "get_"

# Can't do this with def:
# def #{prefix}name; end  # Syntax error!

# Works with define_method:
define_method("#{prefix}name") { @name }
```

Because `def` requires a literal method name, you cannot interpolate variables into it. `define_method` accepts any string or symbol expression.

## Common Pitfalls

1. **Performance in hot paths** — Methods defined with `define_method` are slightly slower than `def` methods because they carry closure overhead. For methods called millions of times, consider using `class_eval` with a string to generate `def`-based methods instead.
2. **Capturing mutable variables** — If the captured variable changes after the method is defined, the method sees the new value. Use `each` (which creates a new binding per iteration) rather than a `while` loop with a counter.

## Best Practices

1. **Use for families of similar methods** — When you have 3+ methods that follow the same pattern and differ only by name or a parameter, `define_method` in a loop is cleaner than copy-pasting.
2. **Document generated methods** — Since dynamically defined methods do not appear literally in the source, add a comment listing the generated method names so developers can find them with search.

## Summary

- `define_method` creates methods at runtime with names that can be computed from data.
- Unlike `def`, the block passed to `define_method` is a closure that captures outer variables.
- It is the tool of choice for generating families of methods from arrays, hashes, or database schemas.

## Code Examples

**Generating getter/setter pairs dynamically from a list of setting names**

```ruby
class Config
  SETTINGS = %i[host port debug]

  SETTINGS.each do |setting|
    define_method(setting) { instance_variable_get("@#{setting}") }
    define_method("#{setting}=") { |v| instance_variable_set("@#{setting}", v) }
  end
end

c = Config.new
c.host = "localhost"
c.port = 3000
puts c.host # => "localhost"
```


## Resources

- [Module#define_method](https://docs.ruby-lang.org/en/4.0/Module.html#method-i-define_method) — Official Ruby documentation for define_method.

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*