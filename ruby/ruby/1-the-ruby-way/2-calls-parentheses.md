---
source_course: "ruby"
source_lesson: "ruby-method-calls-parentheses"
---

# Poetry Mode

Ruby allows you to omit parentheses for method calls. This is idiomatic for DSLs and simple getters, but optional for complex logic.

```ruby
puts "Hello"      # Idiomatic (Poetry Mode)
puts("Hello")     # Valid but noisier
```

## When to Use Parentheses

Use parentheses when:
- There are multiple arguments
- There's ambiguity in parsing
- Chaining method calls

```ruby
# Ambiguous without parens
puts calculate 1, 2  # Error!
puts calculate(1, 2) # Clear

# Chaining
result.map { |x| x * 2 }.sum
```

## Predicate Methods

Methods ending in `?` return a boolean. This is a naming convention, not enforced syntax:

```ruby
[].empty?       # => true
5.positive?     # => true
nil.nil?        # => true
"hello".start_with?("h") # => true
```

## Bang Methods

Methods ending in `!` (bang methods) usually modify the object in place or raise an exception instead of returning nil:

```ruby
name = "hello"
name.upcase!    # modifies name in place
puts name       # => "HELLO"

# Compare:
array = []
array.first     # => nil
array.first!    # raises (if it existed)
```

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*