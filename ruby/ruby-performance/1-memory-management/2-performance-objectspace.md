---
source_course: "ruby-performance"
source_lesson: "ruby-performance-objectspace"
---

# ObjectSpace Module

ObjectSpace allows you to iterate over all live objects in memory - powerful for debugging but expensive.

## Counting Objects by Type

```ruby
require 'objspace'

# Count all String objects
ObjectSpace.each_object(String).count

# Count instances of a specific class
class User; end
5.times { User.new }
ObjectSpace.each_object(User).count  # => 5

# Get object count by type
ObjectSpace.count_objects
# => {:TOTAL=>51548, :FREE=>234, :T_OBJECT=>1287, ...}
```

## Finding Object References

```ruby
require 'objspace'

obj = "hello"

# What objects reference this one?
ObjectSpace.reachable_objects_from(obj)

# Get memory size of an object
ObjectSpace.memsize_of(obj)  # => 40 (bytes)

# Get memsize including referenced objects
ObjectSpace.memsize_of_all(String)  # Total memory of all Strings
```

## Ruby 4.0 Note: _id2ref Deprecated

In Ruby 4.0, `ObjectSpace._id2ref` is deprecated. Avoid using object IDs to retrieve objects:

```ruby
# Deprecated in Ruby 4.0
id = "hello".object_id
ObjectSpace._id2ref(id)  # Warning: deprecated
```

## Resources

- [ObjectSpace](https://docs.ruby-lang.org/en/4.0/ObjectSpace.html) — ObjectSpace module

---

> 📘 *This lesson is part of the [Ruby Internals & Performance](https://stanza.dev/courses/ruby-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*