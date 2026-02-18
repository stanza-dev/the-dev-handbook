---
source_course: "ruby"
source_lesson: "ruby-data-class"
---

# Data Class (Ruby 3.2+)

Ruby 3.2 introduced `Data`, an immutable struct-like class:

```ruby
Point = Data.define(:x, :y)

p = Point.new(10, 20)
p.x  # => 10
p.y  # => 20

# Data objects are immutable
p.x = 30  # NoMethodError!
```

## Data with Pattern Matching

Data classes work beautifully with pattern matching:

```ruby
Result = Data.define(:status, :value)

result = Result.new(:ok, 42)

case result
in Result(status: :ok, value:)
  puts "Success: #{value}"
in Result(status: :error, value: msg)
  puts "Error: #{msg}"
end
```

## Data vs Struct

| Feature | Data | Struct |
|---------|------|--------|
| Mutability | Immutable | Mutable |
| Positional args | Required | Optional |
| Keyword args | Supported | Supported |
| `with` method | Yes | No |

```ruby
# Data's with method for functional updates
Point = Data.define(:x, :y)
p1 = Point.new(1, 2)
p2 = p1.with(x: 10)  # => Point(x: 10, y: 2)
p1.x  # => 1 (original unchanged)
```

## When to Use Data

- Value objects (money, coordinates, etc.)
- Immutable configuration
- API response wrappers
- Anywhere immutability is desired

## Resources

- [Data Class](https://docs.ruby-lang.org/en/4.0/Data.html) — Official Data class documentation

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*