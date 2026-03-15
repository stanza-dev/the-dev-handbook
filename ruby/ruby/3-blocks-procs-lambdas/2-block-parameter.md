---
source_course: "ruby"
source_lesson: "ruby-explicit-block-parameter"
---

# Explicit Block Parameters

## Introduction

While `yield` is great for simple block usage, sometimes you need to store, pass around, or inspect a block. Ruby's `&` operator lets you capture a block as a Proc object, convert Procs back to blocks, and even turn symbols and methods into procs. These conversions are the glue that makes Ruby's closure system so flexible.

## Key Concepts

- **`&block` parameter**: Captures the block passed to a method as a Proc object, allowing you to store it, pass it to other methods, or call it later.
- **Symbol-to-Proc (`&:method_name`)**: The `&` operator calls `to_proc` on a symbol, creating a Proc that sends that method to its argument.
- **Method-to-Proc (`&method(:name)`)**: Converts a named method into a Proc object that can be passed as a block.

## Real World Context

In a callback-heavy system like an event handler or middleware pipeline, you need to store blocks for later execution. For example, a Rack middleware might capture a block as a Proc and call it during request processing. The `&:symbol` shorthand is used constantly in Rails code, like `users.map(&:email)` to extract a list of emails from user objects, saving keystrokes on the most common block pattern.

## Deep Dive

### Capturing Blocks as Procs

Use `&block` in a method signature to capture the block as a Proc object:

```ruby
def capture(&block)
  puts block.class      # => Proc
  block.call            # Execute it
end

capture { puts \"Hello\" }
```

Once captured, the block becomes a first-class Proc that you can store in a variable, pass to other methods, or call multiple times.

### Passing Procs as Blocks

The `&` operator also works in reverse, converting Procs to blocks when calling a method:

```ruby
doubler = Proc.new { |n| n * 2 }
[1, 2, 3].map(&doubler)  # => [2, 4, 6]
```

The `&` tells Ruby to treat the Proc as if it were a block passed to `map`.

### Symbol to Proc

The famous `&:symbol` shorthand converts a symbol to a proc that calls that method on each element:

```ruby
[\"hello\", \"world\"].map(&:upcase)  # => [\"HELLO\", \"WORLD\"]

# Equivalent to:
[\"hello\", \"world\"].map { |s| s.upcase }
```

This works because `&` calls `to_proc` on the symbol, and `Symbol#to_proc` returns a proc that sends the symbol as a message to its argument.

### Method to Proc

You can also convert a named method to a proc using `method`:

```ruby
def double(n)
  n * 2
end

[1, 2, 3].map(&method(:double))  # => [2, 4, 6]
```

This is useful when you have a named method that matches the signature a block expects, letting you reuse existing methods as blocks.

## Common Pitfalls

1. **Using `&block` when `yield` suffices** — Capturing a block with `&block` is slower than `yield` because it allocates a Proc object. Only use `&block` when you actually need to store or forward the block.
2. **Forgetting that `&:method` only works for single-argument methods** — `&:upcase` works because `upcase` takes no arguments (the receiver is the argument). For methods that need additional arguments, you must use a full block: `map { |s| s.gsub('a', 'b') }`.

## Best Practices

1. **Use `&:symbol` for simple single-method blocks** — Replace `.map { |x| x.to_s }` with `.map(&:to_s)`. It is shorter, widely understood, and idiomatic Ruby.
2. **Use `&method(:name)` to reuse existing methods as blocks** — When you already have a method with the right signature, convert it with `&method(:name)` instead of writing a wrapper block.

## Summary

- The `&` operator captures blocks as Procs (`&block`) and converts Procs, symbols, and methods back to blocks.
- `&:symbol` is a powerful shorthand for creating procs that call a single method on each element.
- Use `&block` when you need to store or forward a block; prefer `yield` for simple execution.

## Code Examples

**Demonstrates the & operator for converting between blocks, procs, and symbols**

```ruby
# Symbol to Proc shorthand
["hello", "world"].map(&:upcase)  # => ["HELLO", "WORLD"]

# Capturing a block as a Proc
def capture(&block)
  block.call  # Execute the captured block
end

# Passing a Proc as a block
doubler = Proc.new { |n| n * 2 }
[1, 2, 3].map(&doubler)  # => [2, 4, 6]
```


## Resources

- [Ruby Proc Documentation](https://docs.ruby-lang.org/en/4.0/Proc.html) — Official Ruby 4.0 reference for Proc

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*