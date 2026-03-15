---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-scope-gates"
---

# Scope Gates & Flat Scope

## Introduction

Ruby has three keywords — `class`, `module`, and `def` — that act as scope gates, creating a fresh local-variable scope and blocking access to outer variables. This lesson explains what scope gates are, how to bypass them with the flat-scope technique, and when you actually want the isolation they provide.

## Key Concepts

- **Scope Gate**: A keyword (`class`, `module`, `def`) that opens a new local-variable scope, making outer local variables inaccessible.
- **Flat Scope**: A technique that replaces scope-gate keywords with block-based alternatives (`Class.new`, `define_method`, `class_eval`) to share variables across scopes.
- **Clean Room**: An intentionally isolated scope created with `instance_eval` on a fresh object, preventing access to the caller's state.

## Real World Context

When writing a plugin system, you might need a shared registry variable that is accessible inside dynamically defined methods. The `class` and `def` keywords would block access to that variable. Flat scope lets you use `Class.new` and `define_method` instead, keeping the registry visible. Rails initializers and Rake tasks rely on this pattern to share configuration across method definitions.

## Deep Dive

Three keywords create new scopes, blocking access to outer local variables:
- `class`
- `module`
- `def`

The following example demonstrates the scope gate created by `class`:

```ruby
outer_var = "I'm outside"

class MyClass
  # outer_var is not accessible here!
  puts outer_var  # NameError!
end
```

The `class` keyword opens a new scope, so `outer_var` is invisible inside the class body.

You can flatten the scope by replacing keywords with block-based alternatives that form closures:

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

`Class.new` with a block and `define_method` both use closures, so they retain access to `outer_var`. This is the flat-scope technique.

Here is a practical example where flat scope solves a real problem:

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

The `class_eval` block captures `shared_secret`, and the `define_method` block inside it also retains that reference. The method can now access the variable even though it was defined outside the class.

Sometimes you want the opposite — a completely clean scope that prevents the caller from leaking state:

```ruby
module DSL
  def self.run(&block)
    # New scope, clean environment
    CleanRoom.new.instance_eval(&block)
  end
end
```

The `CleanRoom` object provides a controlled environment where only its own methods are available, preventing accidental access to the caller's variables.

## Common Pitfalls

1. **Overusing flat scope** — Sharing variables across many scopes creates hidden coupling. If a method depends on a captured variable, it becomes harder to understand and test in isolation.
2. **Forgetting that blocks are NOT scope gates** — `do...end` and `{ }` blocks, `if`/`unless`, and `begin`/`rescue` do not create new scopes. Variables defined inside them leak out to the surrounding scope.

## Best Practices

1. **Default to scope gates** — Use `class` and `def` normally. Only reach for flat scope when you have a concrete need to share a variable across boundaries.
2. **Document shared variables** — When using flat scope, comment which variables are captured and why, so future readers understand the coupling.

## Summary

- `class`, `module`, and `def` are scope gates that start fresh local-variable scopes.
- Replace them with `Class.new`, `Module.new`, `class_eval`, and `define_method` to flatten scope and share variables.
- Use clean rooms when you want intentional isolation rather than accidental leakage.

## Code Examples

**Using flat scope to share an environment variable with a dynamically defined method**

```ruby
db_url = ENV.fetch('DATABASE_URL')

# Flat scope: Class.new + define_method share db_url
AppDb = Class.new do
  define_method(:connect) do
    puts "Connecting to #{db_url}"
  end
end

AppDb.new.connect # => "Connecting to postgres://..."
```


## Resources

- [Modules and Classes Syntax](https://docs.ruby-lang.org/en/4.0/syntax/modules_and_classes_rdoc.html) — Official Ruby documentation on class and module syntax, including scoping rules.

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*