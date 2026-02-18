---
source_course: "ruby"
source_lesson: "ruby-regex-captures"
---

# Capturing Groups

Parentheses create capture groups that extract matched portions:

```ruby
match = "John Smith".match(/(\w+) (\w+)/)
match[0]  # => "John Smith" (full match)
match[1]  # => "John" (first group)
match[2]  # => "Smith" (second group)
```

## Named Captures

Ruby supports named captures which map to hash keys:

```ruby
pattern = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/
match = pattern.match("2025-01-19")

match[:year]   # => "2025"
match[:month]  # => "01"
match[:day]    # => "19"
match.named_captures  # => {"year"=>"2025", "month"=>"01", "day"=>"19"}
```

## Automatic Local Variables

Using `=~` with named captures creates local variables:

```ruby
if /(?<name>\w+)@(?<domain>\w+\.\w+)/ =~ "alice@example.com"
  puts name    # => "alice"
  puts domain  # => "example.com"
end
```

## Non-Capturing Groups

Use `(?:...)` when you need grouping but not capturing:

```ruby
# Without capturing
pattern = /(?:https?:\/\/)?(\w+\.\w+)/
match = pattern.match("https://example.com")
match[1]  # => "example.com" (not the protocol!)
```

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*