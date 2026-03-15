---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-refinements"
---

# Refinements

## Introduction

Refinements provide a way to modify classes in a controlled, scoped manner without the global side effects of monkey patching. If you have ever worried about reopening a core class and breaking code elsewhere in your application, refinements are your safety net. This lesson teaches you how to use them effectively so your metaprogramming stays predictable.

## Key Concepts

- **Refinement**: A scoped modification to an existing class that only takes effect where explicitly activated with `using`
- **Lexical scope**: Refinements apply only within the file or module body where `using` is called, not globally
- **`refine` block**: The block inside a module where you define method additions or overrides for a target class

## Real World Context

Imagine you maintain a large Rails application and one gem needs `String#to_bool` while another gem defines the same method differently. With monkey patching, the last definition wins and you get mysterious bugs. With refinements, each file activates only the extensions it needs, and the two definitions never collide.

## Deep Dive

Refinements let you modify classes in a scoped way - changes only apply where you explicitly activate them.

The following example defines a refinement module that adds a `shout` method to `String`:

```ruby
module StringExtensions
  refine String do
    def shout
      upcase + "!"
    end
  end
end

# Without using: no shout method
"hello".shout  # NoMethodError

# Activate refinement
using StringExtensions
"hello".shout  # => "HELLO!"
```

Notice that the `shout` method only becomes available after the `using` statement. Code above that line, or in other files, is unaffected.

### Lexical Scope

Refinements are lexically scoped - they only apply in the file or module body where `using` is called. Here is an example that demonstrates how two files behave differently:

```ruby
# file: extensions.rb
module MyExtensions
  refine Array do
    def second
      self[1]
    end
  end
end

# file: main.rb
require_relative 'extensions'
using MyExtensions

[1, 2, 3].second  # => 2 (works here)

# file: other.rb
require_relative 'extensions'
# No 'using' here
[1, 2, 3].second  # NoMethodError (doesn't affect other files!)
```

The key takeaway is that `other.rb` never calls `using`, so the refinement is invisible there. This is the core safety guarantee.

### Refinement Limitations

There are important constraints to understand. The example below shows that dynamic dispatch does not respect refinements:

```ruby
# Refinements don't affect:
# - Dynamic dispatch (send, method calls via method objects)
# - Kernel#respond_to?
# - Method lookup in other files

using StringExtensions
"hello".send(:shout)  # May not work!
```

This limitation exists because `send` bypasses the lexical scope mechanism that refinements rely on. Always call refined methods directly rather than through dynamic dispatch.



## Common Pitfalls
1. **Expecting refinements to work with `send`** — Refinements are lexically scoped and invisible to dynamic dispatch methods like `send`, `public_send`, and `method_missing`. Always call refined methods directly.
2. **Forgetting the scope boundary** — Refinements activated with `using` only apply from that line forward within the current file or module body. Code loaded via `require` or `load` does not inherit the refinement.

## Best Practices
1. **Prefer refinements over open-class monkey patching** — When you need to add or modify behavior on core classes, refinements limit the blast radius to the file that opts in, preventing global side effects.
2. **Keep refinement modules small and focused** — Each refinement module should add a single, well-defined capability. This makes it easy to understand which behaviors are active in a given file.

## Summary
- Refinements provide lexically scoped monkey patching via `refine` blocks inside modules.
- Activate refinements with `using` — they apply only from that point to the end of the current file or module body.
- Refinements are invisible to `send`, `method_missing`, and other files, which limits unintended side effects.
- Prefer refinements over global monkey patches when you need targeted behavior changes.

## Code Examples

**A practical refinement that adds JSON parsing to String, scoped to a single file**

```ruby
module JSONParsing
  refine String do
    def to_json_obj
      JSON.parse(self)
    end
  end
end

# Only this file gets the method
using JSONParsing
'{"name": "Alice"}'.to_json_obj  # => {"name" => "Alice"}
```


## Resources

- [Refinements Spec](https://docs.ruby-lang.org/en/4.0/syntax/refinements_rdoc.html) — Official Refinements documentation

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*