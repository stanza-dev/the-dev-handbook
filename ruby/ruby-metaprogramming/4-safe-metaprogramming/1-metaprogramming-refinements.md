---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-refinements"
---

# Refinements: Scoped Monkey Patching

Refinements let you modify classes in a scoped way - changes only apply where you explicitly activate them.

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

## Lexical Scope

Refinements are lexically scoped - they only apply in the file/module where `using` is called:

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

## Refinement Limitations

```ruby
# Refinements don't affect:
# - Dynamic dispatch (send, method calls via method objects)
# - Kernel#respond_to?
# - Method lookup in other files

using StringExtensions
"hello".send(:shout)  # May not work!
```

## When to Use Refinements

- **Library extensions** without polluting global namespace
- **Testing helpers** scoped to test files
- **DSLs** that need modified core classes
- **Backward compatibility** polyfills

## Resources

- [Refinements Spec](https://docs.ruby-lang.org/en/4.0/syntax/refinements_rdoc.html) — Official Refinements documentation

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*