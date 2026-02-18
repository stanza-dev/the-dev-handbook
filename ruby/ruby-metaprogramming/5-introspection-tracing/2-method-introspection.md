---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-method-introspection"
---

# Finding Method Origins

Ruby provides rich introspection for methods.

## Method Location

```ruby
class User
  def greet
    "Hello"
  end
end

m = User.new.method(:greet)
m.source_location  # => ["user.rb", 2]
m.owner            # => User
m.receiver         # => #<User:...>

# C methods return nil for source_location
"hello".method(:upcase).source_location  # => nil
```

## Method Parameters

```ruby
def example(required, optional = 1, *rest, keyword:, default_kw: 2, **kwrest, &block)
end

method(:example).parameters
# => [[:req, :required], [:opt, :optional], [:rest, :rest],
#     [:keyreq, :keyword], [:key, :default_kw], [:keyrest, :kwrest],
#     [:block, :block]]
```

## Finding All Methods

```ruby
class Child < Parent
  include SomeModule
end

# Instance methods defined directly on Child
Child.instance_methods(false)

# All instance methods including inherited
Child.instance_methods

# Methods unique to this class (not on Object)
Child.instance_methods - Object.instance_methods

# Private methods
Child.private_instance_methods(false)
```

## Method Defined Checks

```ruby
User.method_defined?(:greet)          # => true
User.method_defined?(:greet, false)   # Only if defined directly
User.instance_method(:greet)          # Get UnboundMethod
```

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*