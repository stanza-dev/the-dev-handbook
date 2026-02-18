---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-method-objects"
---

# Methods as Objects

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

## UnboundMethod

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

## Practical Uses

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
# => nil (C method) or ["file.rb", 42]
```

## Method Introspection

```ruby
m = "hello".method(:gsub)

m.parameters
# => [[:req, :pattern], [:opt, :replacement]]

m.super_method  # => Method for superclass version (if any)
```

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*