---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-activesupport-concern"
---

# ActiveSupport::Concern

Rails' Concern pattern solves common metaprogramming problems.

## The Problem

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

## The Solution

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

## Dependency Resolution

Concern handles dependencies between modules:

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

## Roll Your Own

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

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*