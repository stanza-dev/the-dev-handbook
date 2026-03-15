---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-open-classes"
---

# Open Classes

## Introduction

Ruby lets you reopen any class at any time and add or change its methods. This lesson explores how open classes work, why they are fundamental to Ruby metaprogramming, and when the power they provide is worth the risk.

## Key Concepts

- **Open Class**: A class whose definition can be reopened and modified after its initial creation.
- **Monkey Patching**: Modifying or overriding methods on an existing class, including core Ruby classes.
- **Refinements**: A safer, lexically-scoped alternative to monkey patching introduced in Ruby 2.0.

## Real World Context

Imagine you are integrating a third-party gem that returns plain `String` values, but your app consistently needs a `#to_slug` helper. Rather than wrapping every call, you could reopen `String` and add `#to_slug` once. Rails does exactly this with ActiveSupport core extensions like `"hello".pluralize`.

## Deep Dive

In Ruby, the `class` keyword does not just define a class — it opens the class for modification. If the class already exists, Ruby simply enters its body and executes the code inside.

The following example shows how you can add a new method to the built-in `String` class:

```ruby
class String
  def shout
    upcase + "!"
  end
end

"hello".shout # => "HELLO!"
```

The `shout` method is now available on every `String` instance because Ruby reopened the existing `String` class rather than creating a new one.

You can also reopen your own classes to add methods incrementally:

```ruby
# First definition
class User
  def name
    @name
  end
end

# Later, reopen and add method
class User
  def email
    @email
  end
end

# User now has both methods!
```

Both `name` and `email` are available on `User` instances because the second `class User` block simply reopened the existing class.

Modifying existing classes — especially core classes — is called monkey patching. It is powerful but dangerous because every consumer of that class sees the change:

```ruby
# Dangerous: modifying core behavior
class Array
  def first
    "surprise!"  # Breaks everything!
  end
end
```

Overriding `Array#first` globally means every piece of code in the process that calls `first` on an array will get the wrong result. This is why monkey patching should be used sparingly.

## Common Pitfalls

1. **Overriding core methods silently** — If you redefine a method that already exists (like `Array#first`), Ruby will not warn you. The old implementation is simply replaced, which can break unrelated code.
2. **Gem version conflicts** — Two gems that monkey-patch the same method on the same class will conflict. The last one loaded wins, and the other silently disappears.

## Best Practices

1. **Prefer Refinements** — Use `refine` and `using` to scope your modifications to the file or module that needs them, avoiding global side effects.
2. **Limit to domain-specific extensions** — Only add methods when they represent a genuine concept in your domain (e.g., `String#to_slug` in a CMS), and document them clearly.

## Summary

- Ruby classes can be reopened at any time to add or change methods.
- Monkey patching modifies classes globally and can introduce hard-to-trace bugs.
- Refinements provide a scoped alternative that limits the blast radius of class modifications.

## Code Examples

**Adding a to_slug helper to String via an open class**

```ruby
class String
  def to_slug
    downcase.strip.gsub(/\s+/, '-').gsub(/[^a-z0-9\-]/, '')
  end
end

"Hello World!".to_slug # => "hello-world"
```

**Using Refinements to scope open-class modifications**

```ruby
# Safe alternative using Refinements
module SlugSupport
  refine String do
    def to_slug
      downcase.strip.gsub(/\s+/, '-').gsub(/[^a-z0-9\-]/, '')
    end
  end
end

using SlugSupport
"Hello World!".to_slug # => "hello-world"
```


## Resources

- [Module Class Documentation](https://docs.ruby-lang.org/en/4.0/Module.html) — Official Ruby documentation for the Module class, the foundation of open classes.

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*