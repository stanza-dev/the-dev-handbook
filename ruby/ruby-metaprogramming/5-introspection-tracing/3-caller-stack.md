---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-caller-stack"
---

# Inspecting the Call Stack

Ruby provides several ways to examine the call stack.

## Kernel#caller

```ruby
def deep_method
  caller  # Returns array of strings
end

def middle_method
  deep_method
end

def outer_method
  middle_method
end

outer_method
# => ["example.rb:6:in `middle_method'",
#     "example.rb:10:in `outer_method'",
#     "example.rb:13:in `<main>'"]
```

## caller_locations (More Structured)

```ruby
def show_stack
  caller_locations.each do |loc|
    puts "#{loc.path}:#{loc.lineno} in #{loc.label}"
  end
end

# Returns Thread::Backtrace::Location objects
loc = caller_locations.first
loc.path      # File path
loc.lineno    # Line number
loc.label     # Method name
loc.base_label  # Method name without decoration
```

## Limiting Stack Depth

```ruby
# Only get first 5 frames
caller(0, 5)
caller_locations(0, 5)

# Skip first 2 frames
caller(2)
caller_locations(2)
```

## Practical Uses

```ruby
# Logging with context
def log(message)
  loc = caller_locations(1, 1).first
  puts "[#{loc.path}:#{loc.lineno}] #{message}"
end

# Finding who called a method
def deprecated_method
  warn "Called from: #{caller_locations(1, 1).first}"
end
```

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*