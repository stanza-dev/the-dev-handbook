---
source_course: "ruby"
source_lesson: "ruby-data-class"
---

# Data Classes

## Introduction
Ruby 3.2 introduced `Data`, an immutable alternative to `Struct` designed for value objects. Data classes give you frozen, equality-by-value objects with minimal boilerplate. Combined with pattern matching, they provide a clean way to model domain concepts where immutability matters. This lesson covers `Data.define`, how Data integrates with pattern matching, the differences from Struct, and when to reach for each.

## Key Concepts
- **`Data.define`**: Creates an immutable class with named attributes. Instances are frozen by default.
- **Value Object**: An object defined by its attribute values rather than its identity. Two Data instances with the same values are equal.
- **`with` Method**: Creates a new Data instance with some attributes changed, leaving the original unchanged (functional update).
- **Struct vs Data**: Struct is mutable and flexible; Data is immutable and strict. Choose based on whether you need mutation.

## Real World Context
Value objects appear everywhere in domain-driven design: a `Money` type with `amount` and `currency`, a `Coordinate` with `lat` and `lng`, or an API `Result` with `status` and `value`. Using `Data.define` for these ensures they cannot be accidentally mutated after creation, which prevents entire categories of bugs in concurrent or event-driven systems. The `with` method enables functional transformation patterns — creating new versions of objects rather than mutating existing ones.

## Deep Dive

### Data.define Basics

`Data.define` creates an immutable, frozen class with the specified attributes:

```ruby
Point = Data.define(:x, :y)

p = Point.new(10, 20)
p.x  # => 10
p.y  # => 20

# Data objects are immutable
p.x = 30  # NoMethodError!
```

Unlike Struct, Data instances are frozen immediately upon creation. There are no setter methods, so you cannot accidentally change a value after construction.

### Data with Pattern Matching

Data classes integrate seamlessly with pattern matching, making destructuring clean and type-safe:

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

The `Result(status: :ok, value:)` pattern checks both the type (is it a `Result`?) and the structure (does it have `status: :ok`?). The `value:` shorthand binds the value to a local variable.

### Data vs Struct

The following table summarizes the key differences:

| Feature | Data | Struct |
|---------|------|--------|
| Mutability | Immutable | Mutable |
| Positional args | Required | Optional |
| Keyword args | Supported | Supported |
| `with` method | Yes | No |

The `with` method is Data's killer feature — it creates a copy with specified attributes changed:

```ruby
Point = Data.define(:x, :y)
p1 = Point.new(1, 2)
p2 = p1.with(x: 10)  # => Point(x: 10, y: 2)
p1.x  # => 1 (original unchanged)
```

This enables functional update patterns. Instead of mutating an object, you create a new version, which is safer in concurrent code and easier to reason about.

### When to Use Data

Data classes are ideal for:
- Value objects (money, coordinates, dimensions)
- Immutable configuration objects
- API response wrappers
- Event payloads in event-driven architectures
- Anywhere immutability prevents bugs

## Common Pitfalls
1. **Trying to use Data when you need mutation** — Data objects are frozen. If your use case requires updating attributes in place (e.g., a form builder accumulating fields), use Struct or a regular class instead.
2. **Forgetting that `with` returns a new object** — `p1.with(x: 10)` does not modify `p1`. You must capture the return value: `p2 = p1.with(x: 10)`. This is a common mistake when transitioning from mutable Struct code.

## Best Practices
1. **Default to Data for value objects** — If an object represents a value (coordinates, money, config), reach for `Data.define` first. The immutability guarantee eliminates mutation bugs at the type level.
2. **Combine Data with pattern matching for clean branching** — Define result types like `Success = Data.define(:value)` and `Failure = Data.define(:error)`, then use `case/in` to handle each case. This gives you a lightweight algebraic data type pattern in Ruby.

## Summary
- `Data.define` creates immutable value objects with automatic equality, freezing, and the `with` method for functional updates.
- Data integrates seamlessly with pattern matching for clean destructuring and type-safe branching.
- Prefer Data over Struct when immutability matters; use Struct when you need mutable attributes.

## Code Examples

**Creating immutable Data objects, using with for functional updates, and pattern matching**

```ruby
Point = Data.define(:x, :y)

p1 = Point.new(1, 2)
p2 = p1.with(x: 10)

puts p1  # => Point(x: 1, y: 2)
puts p2  # => Point(x: 10, y: 2)

# Pattern matching with Data
case p2
in Point(x: 10.., y:)
  puts "Far right at y=#{y}"
end
```


## Resources

- [Data Class](https://docs.ruby-lang.org/en/4.0/Data.html) — Official Data class documentation

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*