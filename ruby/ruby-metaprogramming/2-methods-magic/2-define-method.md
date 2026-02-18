---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-define-method"
---

# define_method

Create methods at runtime using `define_method`:

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

## Closures in define_method

Blocks passed to `define_method` are closures:

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

## define_method vs def

| Feature | def | define_method |
|---------|-----|---------------|
| Speed | Faster | Slightly slower |
| Scope | New scope (can't access outer vars) | Closure (captures outer vars) |
| Dynamic name | No | Yes |

```ruby
prefix = "get_"

# Can't do this with def:
# def #{prefix}name; end  # Syntax error!

# Works with define_method:
define_method("#{prefix}name") { @name }
```

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*