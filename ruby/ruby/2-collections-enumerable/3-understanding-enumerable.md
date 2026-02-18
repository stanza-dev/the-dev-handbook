---
source_course: "ruby"
source_lesson: "ruby-understanding-enumerable"
---

# Enumerable

`Enumerable` is a module that gives you `map`, `select`, `reject`, `reduce`, and 50+ other methods for free. It's mixed into Array, Hash, Set, Range, and more.

## Making a Class Enumerable

To make a class Enumerable, you only need to:
1. `include Enumerable`
2. Define an `each` method that yields items.

```ruby
class Team
  include Enumerable

  def initialize(members)
    @members = members
  end

  def each(&block)
    @members.each(&block)
  end
end

team = Team.new(["Alice", "Bob", "Charlie"])
team.map(&:upcase)     # => ["ALICE", "BOB", "CHARLIE"]
team.select { |m| m.start_with?("A") } # => ["Alice"]
```

## Key Enumerable Methods

```ruby
numbers = [1, 2, 3, 4, 5]

# Transformation
numbers.map { |n| n * 2 }     # => [2, 4, 6, 8, 10]
numbers.flat_map { |n| [n, n] } # => [1, 1, 2, 2, ...]

# Filtering
numbers.select { |n| n.even? } # => [2, 4]
numbers.reject { |n| n.even? } # => [1, 3, 5]
numbers.partition { |n| n.even? } # => [[2, 4], [1, 3, 5]]

# Aggregation
numbers.reduce(0) { |sum, n| sum + n } # => 15
numbers.sum                             # => 15 (shortcut)
```

## Resources

- [Enumerable Module](https://docs.ruby-lang.org/en/4.0/Enumerable.html) — Official Enumerable documentation

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*