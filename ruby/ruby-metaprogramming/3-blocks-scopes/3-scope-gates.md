---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-scope-gates"
---

# Scope Gates

Three keywords create new scopes, blocking access to outer variables:
- `class`
- `module`
- `def`

```ruby
outer_var = "I'm outside"

class MyClass
  # outer_var is not accessible here!
  puts outer_var  # NameError!
end
```

## Flattening Scope

Use blocks instead of keywords to maintain scope:

```ruby
outer_var = "I'm outside"

# Instead of class:
MyClass = Class.new do
  puts outer_var  # Works! "I'm outside"
end

# Instead of def:
MyClass.class_eval do
  define_method(:greet) do
    puts outer_var  # Works!
  end
end
```

## Practical Example

```ruby
shared_secret = "password123"

class Authenticator
  # Can't access shared_secret with def
  # def verify(input)
  #   input == shared_secret  # NameError!
  # end
end

# Use flat scope instead
Authenticator.class_eval do
  define_method(:verify) do |input|
    input == shared_secret  # Works!
  end
end
```

## Clean Scope (When You Want Isolation)

Sometimes you want a clean scope:

```ruby
module DSL
  def self.run(&block)
    # New scope, clean environment
    CleanRoom.new.instance_eval(&block)
  end
end
```

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*