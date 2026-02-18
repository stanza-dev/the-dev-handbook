---
source_course: "ruby"
source_lesson: "ruby-array-new-methods"
---

# Array#rfind - Efficient Reverse Search

Ruby 4.0 introduces `Array#rfind`, which is a more efficient alternative to `reverse_each.find` or `reverse.find`.

```ruby
numbers = [1, 2, 3, 4, 5, 4, 3, 2, 1]

# Find last even number
numbers.rfind { |n| n.even? }  # => 2 (the last 2)

# Old way (less efficient)
numbers.reverse.find { |n| n.even? }  # Creates new array!
numbers.reverse_each.find { |n| n.even? } # Better but verbose
```

## Other Useful Array Methods

```ruby
arr = [1, 2, 3, 4, 5]

# Compact - remove nils
[1, nil, 2, nil, 3].compact  # => [1, 2, 3]

# Tally - count occurrences
["a", "b", "a", "c", "b", "a"].tally  # => {"a"=>3, "b"=>2, "c"=>1}

# Intersection and Union
[1, 2, 3] & [2, 3, 4]  # => [2, 3]
[1, 2] | [2, 3]        # => [1, 2, 3]

# Difference
[1, 2, 3] - [2]        # => [1, 3]

# Product - cartesian product
[1, 2].product(["a", "b"])  # => [[1,"a"], [1,"b"], [2,"a"], [2,"b"]]
```

## Lazy Enumerables

For very large collections, use lazy evaluation to avoid creating intermediate arrays:

```ruby
# Without lazy - creates multiple intermediate arrays
(1..Float::INFINITY).select { |n| n.even? }.first(5)
# Error! Infinite loop

# With lazy - processes elements one at a time
(1..Float::INFINITY).lazy.select { |n| n.even? }.first(5)
# => [2, 4, 6, 8, 10]
```

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*