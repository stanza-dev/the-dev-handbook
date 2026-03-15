---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-prepend-patterns"
---

# Prepend Patterns

## Introduction

Prepend is the modern, clean way to wrap existing methods in Ruby without the pitfalls of `alias_method` chaining. When you prepend a module, it is inserted before the class in the ancestor chain, which means the module's methods run first and can call `super` to invoke the original. This lesson covers the technique, its advantages, and practical patterns.

## Key Concepts

- **`prepend`**: Inserts a module before the class in the ancestor chain, so the module's methods override the class's methods
- **Method wrapping**: The pattern of intercepting a method call, adding behavior, and delegating to the original via `super`
- **Anonymous modules**: Modules created with `Module.new` that can be prepended dynamically at runtime

## Real World Context

Suppose you need to add timing, logging, or authorization checks to existing service objects without modifying their source code. Prepend lets you wrap methods cleanly: the original class stays untouched, the wrapper is a separate module, and `super` delegates to the original. This is the pattern behind many Rails middleware and instrumentation libraries.

## Deep Dive

### Basic Wrapper

The fundamental pattern is straightforward. The following example adds timing to a `process` method:

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

Because `Timing` is prepended, it sits before `Worker` in the ancestor chain. When `process` is called, Ruby finds `Timing#process` first, and `super` calls `Worker#process`.

### Dynamic Prepending

You can create and prepend anonymous modules at runtime, which is useful for programmatic method wrapping. Here is a generic logging wrapper:

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

`Module.new` creates a fresh, anonymous module. By using `define_method`, you capture the `method_name` variable in a closure. This approach scales to wrapping any method on any class.

### vs alias_method

Before `prepend` existed, `alias_method` chaining was the standard approach. The comparison below shows why prepend is superior:

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

With `alias_method`, each wrapper adds a new method name to the class, making it hard to stack multiple wrappers or remove them later.

### Multiple Prepends

When you prepend multiple modules, the last one prepended is checked first. This example shows the execution order:

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

The ancestor chain is `[LogB, LogA, Service, ...]`, so `LogB#call` runs first, its `super` calls `LogA#call`, and that `super` calls `Service#call`.

## Common Pitfalls

1. **Forgetting to forward arguments with `super`** — If your prepended method changes the signature, make sure to pass `*args, **kwargs, &block` to `super`. Dropping arguments silently breaks the original method.
2. **Confusing prepend order** — The last module prepended is checked first. If you prepend A then B, the order is B -> A -> Class, not A -> B -> Class.

## Best Practices

1. **Prefer prepend over alias_method for wrapping** — Prepend produces a cleaner ancestor chain, does not pollute the namespace, and stacks predictably when multiple wrappers are applied.
2. **Name your wrapper modules descriptively** — Instead of anonymous modules in production code, use named modules like `Timing` or `Authorization` so they appear clearly in `ancestors` output for debugging.

## Summary

- Prepend inserts a module before the class in the ancestor chain, enabling clean method wrapping via `super`
- It replaces the older `alias_method` chaining pattern with a more predictable and composable approach
- Multiple prepends stack in reverse order: the last module prepended runs first

## Code Examples

**A retry wrapper using prepend that automatically retries failed job execution up to 3 times**

```ruby
module RetryOnFailure
  def perform(*args)
    attempts = 0
    begin
      attempts += 1
      super
    rescue StandardError => e
      retry if attempts < 3
      raise
    end
  end
end

class EmailJob
  prepend RetryOnFailure

  def perform(to:, subject:)
    # send email, may raise
  end
end
```

**An authorization wrapper that gates destructive operations behind an admin check**

```ruby
module Authorization
  def delete(*args)
    raise "Unauthorized" unless current_user&.admin?
    super
  end
end

class PostService
  prepend Authorization

  def delete(post_id)
    Post.find(post_id).destroy
  end
end
```


## Resources

- [Module#prepend](https://docs.ruby-lang.org/en/4.0/Module.html#method-i-prepend) — Official documentation for Module#prepend covering method resolution and ancestor chain insertion

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*