---
source_course: "ruby"
source_lesson: "ruby-everything-is-an-object"
---

# Everything is an Object

In Ruby, there are no primitives. Every value is an instance of a class. `1` is an instance of `Integer`. `true` is an instance of `TrueClass`. Even `nil` is an object!

```ruby
1.class          # => Integer
true.class       # => TrueClass
nil.class        # => NilClass
"hello".class    # => String
```

## Calling Methods on Literals

Because everything is an object, you can call methods directly on literals:

```ruby
1.methods.first  # => :%
-1.abs           # => 1
42.even?         # => true
3.14.round       # => 3
```

## The `nil` Object

Even `nil` is an object (of class `NilClass`). It is the only "falsy" value besides `false`.

```ruby
nil.nil?   # => true
nil.to_s   # => ""
nil.to_a   # => []
0.nil?     # => false (0 is truthy in Ruby!)
```

## The Object Hierarchy

All objects ultimately inherit from `BasicObject`, through `Object` and `Kernel`:

```ruby
Integer.ancestors
# => [Integer, Numeric, Comparable, Object, Kernel, BasicObject]
```

## Resources

- [Object Class](https://docs.ruby-lang.org/en/4.0/Object.html) — The root of the Ruby class hierarchy.

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*