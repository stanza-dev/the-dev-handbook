---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-context-probe"
---

# Introspecting Context

Ruby provides many ways to examine the current execution context.

## What's self?

```ruby
class Demo
  puts self        # => Demo

  def instance_method
    puts self      # => #<Demo:0x...>
  end

  class << self
    puts self      # => #<Class:Demo>
  end
end
```

## Local Variables

```ruby
a = 1
b = 2
local_variables  # => [:a, :b]

binding.local_variable_get(:a)  # => 1
```

## Instance Variables

```ruby
@x = 10
@y = 20
instance_variables  # => [:@x, :@y]

instance_variable_get(:@x)  # => 10
instance_variable_set(:@z, 30)
```

## Methods

```ruby
class Example
  def public_method; end
  private
  def private_method; end
end

Example.instance_methods(false)         # => [:public_method]
Example.private_instance_methods(false) # => [:private_method]

# On an instance
obj = Example.new
obj.methods - Object.methods  # Methods unique to Example
```

## Constants

```ruby
module Container
  CONST_A = 1
  class Inner; end
end

Container.constants  # => [:CONST_A, :Inner]
Container.const_get(:CONST_A)  # => 1
```

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*