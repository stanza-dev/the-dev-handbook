---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-prepend-patterns"
---

# Method Wrapping with Prepend

Prepend is the cleanest way to wrap existing methods.

## Basic Wrapper

```ruby
module Timing
  def process(*args)
    start = Time.now
    result = super  # Call original
    puts "Took #{Time.now - start}s"
    result
  end
end

class Worker
  prepend Timing

  def process(data)
    sleep(0.1)
    data.upcase
  end
end

Worker.new.process("hello")
# "Took 0.1s"
# => "HELLO"
```

## Dynamic Prepending

```ruby
def add_logging(klass, method_name)
  wrapper = Module.new do
    define_method(method_name) do |*args, &block|
      puts "Calling #{method_name}"
      super(*args, &block)
    end
  end
  klass.prepend(wrapper)
end

add_logging(String, :upcase)
"hello".upcase
# "Calling upcase"
# => "HELLO"
```

## vs alias_method

```ruby
# Old way (alias_method chain)
class String
  alias_method :original_upcase, :upcase

  def upcase
    puts "Calling upcase"
    original_upcase
  end
end

# Problems:
# - Pollutes namespace with original_*
# - Ordering issues with multiple wrappers
# - Can't easily remove

# Prepend is cleaner and more predictable
```

## Multiple Prepends

```ruby
module LogA
  def call; puts "A"; super; end
end

module LogB
  def call; puts "B"; super; end
end

class Service
  prepend LogA
  prepend LogB  # LogB comes first now

  def call; puts "Service"; end
end

Service.new.call
# B
# A
# Service
```

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*