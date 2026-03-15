---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-ancestor-chain"
---

# The Ancestor Chain

## Introduction

Every method call in Ruby triggers a lookup walk through the ancestor chain. Understanding this chain is essential for predicting which implementation of a method will run, especially when modules, inheritance, and prepend are all in play.

## Key Concepts

- **Ancestor Chain**: The ordered list of classes and modules Ruby searches when resolving a method call.
- **include**: Inserts a module into the ancestor chain after the class.
- **prepend**: Inserts a module into the ancestor chain before the class, so it gets first shot at method calls.

## Real World Context

When a Rails controller action fires, the request passes through multiple modules (authentication, logging, parameter parsing) that are mixed into the controller hierarchy. Knowing the ancestor chain lets you predict the order those layers execute and debug cases where the wrong implementation wins.

## Deep Dive

When you call a method, Ruby searches for it in a specific order:

1. The **Singleton Class** (if it exists)
2. **Prepended Modules** (in reverse prepend order)
3. The **Class** itself
4. **Included Modules** (in reverse inclusion order)
5. The **Superclass**
6. Repeat steps 2-5 up the superclass chain
7. `Object` → `Kernel` → `BasicObject`

The following example shows how `include` inserts modules into the chain:

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

Notice that `B` appears right after `Child` (where it was included), and `A` appears after `Parent`.

`prepend` works differently from `include` — it places the module before the class in the chain, so the module's methods are found first:

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

Because `Logging` is prepended, it appears before `Record` in the chain. When `save` is called, Ruby finds `Logging#save` first, and `super` then delegates to `Record#save`.

The following example demonstrates how `super` chains through multiple levels of the ancestor hierarchy:

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

Ruby resolves `greet` on `Child` first, then `super` goes to `Mixin#greet`, and its `super` reaches `Parent#greet`.

## Common Pitfalls

1. **Forgetting that include order matters** — Modules included later appear earlier in the chain. If you `include A` then `include B`, the chain is `[Class, B, A, ...]`, not `[Class, A, B, ...]`.
2. **Confusing prepend and include** — `prepend` inserts before the class; `include` inserts after. Mixing them up leads to methods that never fire or fire in the wrong order.

## Best Practices

1. **Inspect with `.ancestors`** — When in doubt, call `MyClass.ancestors` in a console to see the exact lookup order Ruby will follow.
2. **Use prepend for method wrapping** — When you need to wrap an existing method (logging, timing, validation), `prepend` is cleaner than `alias_method` chains because it uses `super` naturally.

## Summary

- Ruby resolves methods by walking the ancestor chain from the object's singleton class up to `BasicObject`.
- `include` inserts modules after the class; `prepend` inserts them before it.
- Calling `.ancestors` on any class reveals the exact lookup order Ruby will use.

## Code Examples

**Using prepend to wrap a method via the ancestor chain**

```ruby
module Auditable
  def save
    puts "Audit: saving #{self.class}"
    super
  end
end

class Record
  prepend Auditable
  def save
    puts "Record saved"
  end
end

Record.ancestors # => [Auditable, Record, Object, Kernel, BasicObject]
Record.new.save  # Audit: saving Record\nRecord saved
```


## Resources

- [Module#ancestors](https://docs.ruby-lang.org/en/4.0/Module.html#method-i-ancestors) — Official Ruby documentation for the ancestors method that returns the ancestor chain.

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*