---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-activesupport-concern"
---

# The Concern Pattern

## Introduction

ActiveSupport::Concern is Rails' answer to the boilerplate that comes with using `included` hooks and nested `ClassMethods` modules. It simplifies the pattern so much that it has become the standard way to write mixins in Rails applications. This lesson shows you the problem it solves and how to use it — and even how to build a minimal version yourself.

## Key Concepts

- **Concern**: A module that extends `ActiveSupport::Concern` to gain a cleaner DSL for hooks and class methods
- **`included` block**: A block (not a method) that runs in the context of the including class
- **`class_methods` block**: Replaces the manual `module ClassMethods` + `base.extend` pattern
- **Dependency resolution**: Concerns automatically handle chains of `include` so that `included` blocks run in the correct order

## Real World Context

In a typical Rails application you might have dozens of shared behaviors — Searchable, Taggable, Auditable, SoftDeletable. Without Concern, every one of these modules requires the same `self.included(base)` boilerplate. With Concern, each module is concise, readable, and correctly handles dependency chains when one concern includes another.

## Deep Dive

### The Problem

Without Concern, adding class methods and running class-level code from a module requires verbose boilerplate. Here is the traditional approach:

```ruby
# Without Concern - verbose and error-prone
module Taggable
  def self.included(base)
    base.extend(ClassMethods)
    base.class_eval do
      has_many :tags
    end
  end

  module ClassMethods
    def most_tagged
      # ...
    end
  end

  def tag_names
    tags.map(&:name)
  end
end
```

Every module that needs class methods repeats this exact same structure, which is tedious and error-prone.

### The Solution

ActiveSupport::Concern wraps this boilerplate into a clean DSL:

```ruby
require 'active_support/concern'

module Taggable
  extend ActiveSupport::Concern

  included do
    has_many :tags
  end

  class_methods do
    def most_tagged
      # ...
    end
  end

  def tag_names
    tags.map(&:name)
  end
end
```

The `included` block replaces `self.included(base)` plus `base.class_eval`, and `class_methods` replaces the nested `ClassMethods` module plus `base.extend`. The result is more readable and less error-prone.

### Dependency Resolution

Concern also handles dependencies between modules correctly. When one concern includes another, the `included` blocks run in the right order:

```ruby
module A
  extend ActiveSupport::Concern
  included do
    # This runs when A is included
  end
end

module B
  extend ActiveSupport::Concern
  include A  # A's included block runs when B is included
end

class MyClass
  include B  # Both A and B's included blocks run
end
```

Without Concern, including A inside B would immediately trigger A's `included` hook with B as the base, which is usually not what you want. Concern defers execution until the final class includes the outermost module.

### Roll Your Own

To understand the mechanics, here is a simplified version of Concern:

```ruby
module SimpleConcern
  def self.extended(base)
    base.instance_variable_set(:@_included_block, nil)
  end

  def included(base = nil, &block)
    if block
      @_included_block = block
    else
      base.class_eval(&@_included_block) if @_included_block
    end
  end
end
```

This stores the block and replays it when the module is finally included into a class. The real ActiveSupport implementation adds dependency tracking and class method support.

## Common Pitfalls

1. **Forgetting `extend ActiveSupport::Concern`** — Without this line, `included do ... end` defines a regular method called `included` instead of registering a callback block. Your code will silently do nothing.
2. **Using `self.included` instead of the `included` block** — When using Concern, always use the block form. Defining `self.included` overrides the Concern mechanism and breaks dependency resolution.

## Best Practices

1. **One responsibility per concern** — Each concern module should encapsulate a single behavior (e.g., Taggable, Searchable). Combining unrelated behaviors in one concern defeats the purpose.
2. **Use `class_methods` instead of `ClassMethods` module** — The block-based `class_methods` DSL is cleaner and avoids the manual `extend` step. It also integrates with Concern's dependency resolution.

## Summary

- ActiveSupport::Concern removes boilerplate from the `included` + `ClassMethods` pattern
- It provides `included` and `class_methods` blocks that run in the correct context automatically
- Dependency resolution ensures that chained concerns execute their hooks in the right order

## Code Examples

**A Searchable concern that adds scoped search, class methods, and instance methods to any ActiveRecord model**

```ruby
require 'active_support/concern'

module Searchable
  extend ActiveSupport::Concern

  included do
    scope :search, ->(query) { where("name LIKE ?", "%#{query}%") }
  end

  class_methods do
    def search_fields
      [:name, :description]
    end
  end

  def matching_terms(query)
    self.class.search_fields.select { |f| send(f)&.include?(query) }
  end
end
```


## Resources

- [ActiveSupport::Concern](https://api.rubyonrails.org/classes/ActiveSupport/Concern.html) — Official Rails documentation for ActiveSupport::Concern with examples and usage patterns

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*