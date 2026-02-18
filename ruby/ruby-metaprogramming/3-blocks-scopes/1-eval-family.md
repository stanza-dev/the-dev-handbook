---
source_course: "ruby-metaprogramming"
source_lesson: "ruby-metaprogramming-eval-family"
---

# instance_eval

Evaluates a block with `self` set to the receiver object. You can access private methods and instance variables:

```ruby
class Person
  def initialize
    @secret = "hidden"
  end
end

person = Person.new

# Access private state
person.instance_eval { @secret }  # => "hidden"

# Add singleton method
person.instance_eval do
  def speak
    "I know #{@secret}"
  end
end
```

## class_eval / module_eval

Evaluates code in the context of a class/module. Used to add methods dynamically:

```ruby
User.class_eval do
  def greet
    "Hello!"
  end
end

User.new.greet  # => "Hello!"
```

## Key Difference

```ruby
class User; end

# instance_eval: defines SINGLETON methods (class methods)
User.instance_eval do
  def find(id); end  # User.find(1) works
end

# class_eval: defines INSTANCE methods
User.class_eval do
  def save; end      # User.new.save works
end
```

## DSL Pattern

```ruby
class ConfigBuilder
  def initialize(&block)
    instance_eval(&block) if block
  end

  def port(n)
    @port = n
  end

  def host(h)
    @host = h
  end
end

config = ConfigBuilder.new do
  port 8080
  host "localhost"
end
```

---

> 📘 *This lesson is part of the [Ruby Metaprogramming Workshop](https://stanza.dev/courses/ruby-metaprogramming) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*