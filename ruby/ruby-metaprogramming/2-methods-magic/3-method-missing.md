---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-method-missing"
---

# method_missing

When Ruby can't find a method, it calls `method_missing` as a last resort:

```ruby
class Ghost
  def method_missing(name, *args)
    puts "You called #{name} with #{args}"
  end
end

g = Ghost.new
g.anything(1, 2, 3)  # "You called anything with [1, 2, 3]"
```

## Practical Example: Dynamic Finders

```ruby
class User
  def self.method_missing(name, *args)
    if name.to_s.start_with?("find_by_")
      attribute = name.to_s.sub("find_by_", "")
      find_by_attribute(attribute, args.first)
    else
      super  # Important: call super if not handled!
    end
  end

  private

  def self.find_by_attribute(attr, value)
    # Database query...
    "Finding by #{attr} = #{value}"
  end
end

User.find_by_name("Alice")   # Works!
User.find_by_email("a@b.c")  # Works!
```

## Critical Rule: respond_to_missing?

**Always** implement `respond_to_missing?` alongside `method_missing`:

```ruby
class Ghost
  def method_missing(name, *args)
    # ...
  end

  def respond_to_missing?(name, include_private = false)
    name.to_s.start_with?("find_by_") || super
  end
end

# Without respond_to_missing?:
g.respond_to?(:find_by_x)  # => false (wrong!)

# With respond_to_missing?:
g.respond_to?(:find_by_x)  # => true (correct!)
```

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*