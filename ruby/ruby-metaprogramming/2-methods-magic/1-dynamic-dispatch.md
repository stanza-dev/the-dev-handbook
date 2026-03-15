---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-dynamic-dispatch"
---

# Dynamic Dispatch with Send

## Introduction

Ruby's `send` method lets you call any method by name at runtime, turning method names into data you can store, compute, and pass around. This lesson covers dynamic dispatch with `send` and `public_send`, and when each is appropriate.

## Key Concepts

- **Dynamic Dispatch**: Deciding which method to call at runtime rather than at write time.
- **send**: Calls a method by name (symbol or string), bypassing visibility checks.
- **public_send**: Like `send`, but respects `public`/`private`/`protected` visibility.

## Real World Context

Consider a REST API controller that maps incoming JSON attributes to setter methods on a model. Instead of writing a giant `case` statement, you can iterate over the attribute hash and call `user.public_send("#{key}=", value)` for each key. ActiveRecord's `assign_attributes` uses exactly this pattern.

## Deep Dive

You can call any method by name using `send(symbol, args)`:

```ruby
obj = "hello"
obj.send(:upcase)          # => "HELLO"
obj.send(:[], 1)           # => "e" (same as obj[1])
obj.send(:gsub, "l", "L")  # => "heLLo"
```

The first argument is the method name as a symbol (or string), followed by the arguments to pass. This is equivalent to calling the method directly, but the name can be dynamic.

Dynamic method selection becomes powerful when the method name comes from data:

```ruby
class Calculator
  def add(a, b); a + b; end
  def sub(a, b); a - b; end
  def mul(a, b); a * b; end
end

calc = Calculator.new
operation = :add  # Could come from user input

calc.send(operation, 10, 5)  # => 15
```

The `operation` variable determines which method runs, enabling flexible dispatch without conditionals.

`send` can call private methods, which breaks encapsulation. Use `public_send` when you want to respect visibility:

```ruby
class Secret
  private
  def password; "12345"; end
end

s = Secret.new
s.send(:password)        # => "12345" (works!)
s.public_send(:password) # NoMethodError (private method)
```

`public_send` raises `NoMethodError` for private methods, just like a normal method call would. This makes it the safer default.

Here are two practical applications of dynamic dispatch:

```ruby
# Calling setters dynamically
attrs = { name: "Alice", age: 30 }
attrs.each { |key, value| user.send("#{key}=", value) }

# Testing private methods
it "calculates internally" do
  expect(obj.send(:private_calc)).to eq(42)
end
```

The first example iterates over a hash and calls the corresponding setter for each key. The second uses `send` in tests to reach private methods — a common testing pattern, though controversial.

## Common Pitfalls

1. **Using `send` with unsanitized user input** — If the method name comes from user input, an attacker could call `send(:exit!)` or `send(:system, "rm -rf /")`. Always whitelist allowed method names or use `public_send`.
2. **Forgetting that `send` bypasses private** — Using `send` when `public_send` would suffice makes your code less safe and harder to reason about.

## Best Practices

1. **Default to `public_send`** — Only use `send` when you specifically need to bypass visibility, such as in testing or framework internals.
2. **Whitelist dynamic method names** — When dispatching based on external input, validate the method name against an explicit list of allowed values.

## Summary

- `send` calls any method by name at runtime, including private methods.
- `public_send` respects visibility and is the safer choice for most use cases.
- Dynamic dispatch eliminates repetitive conditionals but requires careful input validation.

## Code Examples

**Whitelisting method names before dynamic dispatch for safety**

```ruby
ALLOWED = %i[add sub mul]

def safe_dispatch(calc, operation, a, b)
  raise ArgumentError, "Unknown op" unless ALLOWED.include?(operation)
  calc.public_send(operation, a, b)
end

safe_dispatch(Calculator.new, :add, 10, 5) # => 15
```


## Resources

- [Object#send](https://docs.ruby-lang.org/en/4.0/Object.html#method-i-send) — Documentation for send

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*