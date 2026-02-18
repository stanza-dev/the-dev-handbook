---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-dynamic-dispatch"
---

# send: Call Methods by Name

You can call any method by name using `send(symbol, args)`:

```ruby
obj = "hello"
obj.send(:upcase)          # => "HELLO"
obj.send(:[], 1)           # => "e" (same as obj[1])
obj.send(:gsub, "l", "L")  # => "heLLo"
```

## Dynamic Method Selection

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

## send vs public_send

`send` can call private methods. Use `public_send` for safety:

```ruby
class Secret
  private
  def password; "12345"; end
end

s = Secret.new
s.send(:password)        # => "12345" (works!)
s.public_send(:password) # NoMethodError (private method)
```

## Practical Uses

```ruby
# Calling setters dynamically
attrs = { name: "Alice", age: 30 }
attrs.each { |key, value| user.send("#{key}=", value) }

# Testing private methods
it "calculates internally" do
  expect(obj.send(:private_calc)).to eq(42)
end
```

## Resources

- [Object#send](https://docs.ruby-lang.org/en/4.0/Object.html#method-i-send) — Documentation for send

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*