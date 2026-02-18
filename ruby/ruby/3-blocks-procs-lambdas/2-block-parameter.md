---
source_course: "ruby"
source_lesson: "ruby-explicit-block-parameter"
---

# Capturing Blocks as Procs

Use `&block` to capture a block as a Proc object:

```ruby
def capture(&block)
  puts block.class      # => Proc
  block.call            # Execute it
end

capture { puts "Hello" }
```

## Passing Procs as Blocks

The `&` operator also converts Procs to blocks:

```ruby
doubler = Proc.new { |n| n * 2 }
[1, 2, 3].map(&doubler)  # => [2, 4, 6]
```

## Symbol to Proc

The famous `&:symbol` shorthand converts a symbol to a proc that calls that method:

```ruby
["hello", "world"].map(&:upcase)  # => ["HELLO", "WORLD"]

# Equivalent to:
["hello", "world"].map { |s| s.upcase }
```

## Method to Proc

You can also convert a method to a proc:

```ruby
def double(n)
  n * 2
end

[1, 2, 3].map(&method(:double))  # => [2, 4, 6]
```

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*