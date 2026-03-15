---
source_course: "ruby"
source_lesson: "ruby-regex-captures"
---

# Capturing Groups & Named Captures

## Introduction
Capturing groups let you extract specific portions of a matched string. Named captures take this further by assigning meaningful labels to each group. These features turn regex from a simple matching tool into a powerful data extraction engine.

## Key Concepts
- **Capture group**: Parentheses `()` in a regex that save the matched substring for later access via index.
- **Named capture**: A capture group with a label, written as `(?<name>...)`, accessible by name instead of position.
- **MatchData**: The object returned by `match` that holds the full match and all captured groups.
- **Non-capturing group**: `(?:...)` groups for precedence without saving the matched text.

## Real World Context
Parsing log files is a daily task for backend developers. A log line like `[2025-01-19 14:30:05] ERROR: Connection refused` contains a timestamp, a severity level, and a message. Named captures let you extract each piece into a readable hash with one regex, instead of writing a multi-step string splitting pipeline.

## Deep Dive

### Capturing Groups

Parentheses create capture groups that extract matched portions. Each group is numbered left-to-right starting at 1, while index 0 holds the full match:

```ruby
match = "John Smith".match(/(\w+) (\w+)/)
match[0]  # => "John Smith" (full match)
match[1]  # => "John" (first group)
match[2]  # => "Smith" (second group)
```

Numbered captures work well for simple patterns, but become hard to read when a regex has more than two or three groups.

### Named Captures

Ruby supports named captures which map to hash-style access on the `MatchData` object. This makes your code self-documenting:

```ruby
pattern = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/
match = pattern.match("2025-01-19")

match[:year]   # => "2025"
match[:month]  # => "01"
match[:day]    # => "19"
match.named_captures  # => {"year"=>"2025", "month"=>"01", "day"=>"19"}
```

The `named_captures` method returns a plain `Hash`, which is convenient for passing extracted data to other methods.

### Automatic Local Variables

A unique Ruby feature: when you use `=~` with a regex literal containing named captures on the left side, Ruby automatically creates local variables for each capture:

```ruby
if /(?<name>\w+)@(?<domain>\w+\.\w+)/ =~ "alice@example.com"
  puts name    # => "alice"
  puts domain  # => "example.com"
end
```

This only works when the regex literal is on the left side of `=~`. If the string is on the left, or the regex is stored in a variable, the automatic variable creation does not happen.

### Non-Capturing Groups

Use `(?:...)` when you need grouping for alternation or quantifiers but do not want to save the matched text:

```ruby
# Without capturing the protocol
pattern = /(?:https?:\/\/)?(\w+\.\w+)/
match = pattern.match("https://example.com")
match[1]  # => "example.com" (not the protocol!)
```

Non-capturing groups keep your capture indices clean and avoid unnecessary `MatchData` entries, which matters when patterns grow complex.

## Common Pitfalls
1. **Relying on numbered captures in complex patterns** — When a regex has many groups, it is easy to lose track of which index corresponds to which piece of data. Named captures are more maintainable.
2. **Expecting automatic variables from =~ with a variable** — The automatic local variable feature only works with a regex literal on the left side of `=~`. Using a regex stored in a variable silently skips variable creation.

## Best Practices
1. **Prefer named captures for readability** — Named captures make your regex self-documenting and your extraction code resistant to changes in group ordering.
2. **Use named_captures to get a Hash** — Calling `match.named_captures` gives you a plain `Hash` that integrates cleanly with the rest of your code.

## Summary
- Capture groups `()` extract matched substrings, accessible by index on the `MatchData` object.
- Named captures `(?<name>...)` let you access groups by name, which is more readable and maintainable.
- Use non-capturing groups `(?:...)` when you need grouping without extraction.

## Code Examples

**Demonstrates parsing structured log lines using named captures and extracting fields into a hash.**

```ruby
# Parsing a log line with named captures
log_line = "[2025-01-19 14:30:05] ERROR: Connection refused"

pattern = /\[(?<date>[\d-]+) (?<time>[\d:]+)\] (?<level>\w+): (?<message>.+)/
if m = log_line.match(pattern)
  puts m[:date]    # => 2025-01-19
  puts m[:time]    # => 14:30:05
  puts m[:level]   # => ERROR
  puts m[:message] # => Connection refused
  puts m.named_captures
  # => {"date"=>"2025-01-19", "time"=>"14:30:05", "level"=>"ERROR", "message"=>"Connection refused"}
end
```


## Resources

- [Regexp Class](https://docs.ruby-lang.org/en/4.0/Regexp.html) — Official Ruby documentation for Regexp, including capture groups and named captures.

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*