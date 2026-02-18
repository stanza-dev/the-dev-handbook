---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-ancestor-chain"
---

# Method Lookup

When you call a method, Ruby searches for it in a specific order:

1. The **Singleton Class** (if it exists)
2. The **Class** itself
3. **Prepended Modules** (in reverse order)
4. **Included Modules** (in reverse inclusion order)
5. The **Superclass**
6. Repeat 3-5 for superclass
7. `Object` → `Kernel` → `BasicObject`

```ruby
module A; end
module B; end

class Parent
  include A
end

class Child < Parent
  include B
end

Child.ancestors
# => [Child, B, Parent, A, Object, Kernel, BasicObject]
```

## Prepend vs Include

```ruby
module Logging
  def save
    puts "Logging..."
    super  # Calls the class method
  end
end

class Record
  prepend Logging  # Logging comes BEFORE Record

  def save
    puts "Saving"
  end
end

Record.ancestors  # [Logging, Record, Object, Kernel, BasicObject]
Record.new.save   # "Logging..." then "Saving"
```

## Method Resolution Demo

```ruby
class Parent
  def greet; "parent"; end
end

module Mixin
  def greet; "mixin: " + super; end
end

class Child < Parent
  include Mixin
  def greet; "child: " + super; end
end

Child.new.greet  # => "child: mixin: parent"
```

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*