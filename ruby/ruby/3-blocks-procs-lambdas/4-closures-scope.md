---
source_course: "ruby"
source_lesson: "ruby-closures-scope"
---

# Closures Capture Their Environment

Blocks, Procs, and Lambdas are closures - they capture variables from their surrounding scope:

```ruby
def counter
  count = 0
  -> { count += 1 }  # Lambda captures 'count'
end

c = counter
c.call  # => 1
c.call  # => 2
c.call  # => 3
```

## Sharing State Between Closures

```ruby
def create_counter
  count = 0
  {
    increment: -> { count += 1 },
    decrement: -> { count -= 1 },
    value: -> { count }
  }
end

counter = create_counter
counter[:increment].call  # => 1
counter[:increment].call  # => 2
counter[:decrement].call  # => 1
counter[:value].call      # => 1
```

## The Binding Object

Every closure has a `binding` that captures its execution context:

```ruby
def get_binding(value)
  binding
end

b = get_binding(42)
b.local_variable_get(:value)  # => 42
b.eval("value * 2")           # => 84
```

## Warning: Closures Keep References

Closures hold references to captured variables, which can cause unexpected behavior:

```ruby
lambdas = []
3.times do |i|
  lambdas << -> { i }
end
lambdas.map(&:call)  # => [0, 1, 2] - each captured its own 'i'
```

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*