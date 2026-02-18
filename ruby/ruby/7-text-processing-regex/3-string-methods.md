---
source_course: "ruby"
source_lesson: "ruby-string-methods"
---

# String Manipulation Methods

## Search & Replace

```ruby
text = "Hello World, Hello Ruby"

# Replace first occurrence
text.sub("Hello", "Hi")     # => "Hi World, Hello Ruby"

# Replace all occurrences
text.gsub("Hello", "Hi")    # => "Hi World, Hi Ruby"

# With regex and captures
"john_doe".gsub(/_(.)/){ $1.upcase }  # => "johnDoe"
```

## Scanning All Matches

```ruby
"ruby 2.7, ruby 3.0, ruby 4.0".scan(/ruby (\d+\.\d+)/)
# => [["2.7"], ["3.0"], ["4.0"]]

# Just the versions
"ruby 2.7, ruby 3.0, ruby 4.0".scan(/\d+\.\d+/)
# => ["2.7", "3.0", "4.0"]
```

## Ruby 4.0: String#strip with Selectors

Ruby 4.0 enhances `strip`, `lstrip`, and `rstrip` to accept character selectors:

```ruby
# Traditional - removes whitespace
"  hello  ".strip  # => "hello"

# Ruby 4.0 - specify what to strip
"###hello###".strip("#")     # => "hello"
"...hello...".lstrip(".")    # => "hello..."
"000123".lstrip("0")          # => "123"

# Multiple characters
"<>hello<>".strip("<>")      # => "hello"
```

## Splitting

```ruby
"a,b,c".split(",")           # => ["a", "b", "c"]
"a  b  c".split              # => ["a", "b", "c"] (splits on whitespace)
"a1b2c3".split(/\d/)         # => ["a", "b", "c"]
```

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*