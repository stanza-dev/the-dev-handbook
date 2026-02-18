---
source_course: "ruby"
source_lesson: "ruby-set-core-class"
---

# Set is Now a Core Class

In Ruby 4.0, `Set` has been promoted from the standard library to a **core class**. No more `require 'set'` needed!

```ruby
# Ruby 4.0 - no require needed!
fruits = Set.new(["apple", "banana", "apple"])
puts fruits # => #<Set: {"apple", "banana"}>
```

## Set Characteristics

- **Unordered collection of unique values**
- Fast membership testing (`O(1)` lookup)
- Automatically deduplicates entries

```ruby
set = Set.new([1, 2, 2, 3, 3, 3])
set.to_a  # => [1, 2, 3]

# Adding elements
set.add(4)     # => Set adds 4
set << 5       # => Same as add
set.add(1)     # => No effect, 1 already exists
```

## Common Set Operations

```ruby
a = Set[1, 2, 3]
b = Set[2, 3, 4]

a | b          # Union: Set[1, 2, 3, 4]
a & b          # Intersection: Set[2, 3]
a - b          # Difference: Set[1]
a ^ b          # Symmetric difference: Set[1, 4]

a.subset?(b)   # => false
a.superset?(Set[1, 2]) # => true
```

## When to Use Set vs Array

| Use Set when | Use Array when |
|--------------|----------------|
| You need unique values | Order matters |
| Frequent membership checks | You need duplicates |
| Set operations (union, etc.) | Index-based access |

## Resources

- [Set Class](https://docs.ruby-lang.org/en/4.0/Set.html) — Official Set documentation

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*