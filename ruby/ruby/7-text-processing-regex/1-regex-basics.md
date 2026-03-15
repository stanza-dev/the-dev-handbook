---
source_course: "ruby"
source_lesson: "ruby-regex-basics"
---

# Regular Expressions

## Introduction
Ruby has first-class support for regular expressions using the `/pattern/` literal syntax. Regex is essential for validating input, extracting data from text, and performing complex search-and-replace operations. Ruby's Regexp integration is one of the most ergonomic in any programming language.

## Key Concepts
- **Regexp literal**: A pattern enclosed in forward slashes like `/hello/`. Ruby compiles this into a `Regexp` object.
- **=~ operator**: Returns the index of the first match (or `nil`), and is commonly used in conditionals.
- **match?**: Returns a boolean without creating a `MatchData` object, making it the fastest way to check for a match.
- **Character classes**: Sets of characters enclosed in brackets like `[aeiou]` that match any single character in the set.

## Real World Context
A web application receives user input for email addresses, phone numbers, and dates. Before storing this data, you need to validate its format. Regex lets you express these validation rules concisely. For example, checking that a string looks like an email takes one line of regex instead of dozens of lines of manual string parsing.

## Deep Dive

### Basic Matching

Ruby provides three main ways to test a string against a regex pattern. The `=~` operator returns the match position:

```ruby
text = "Hello World"

# Using =~
if text =~ /World/
  puts "Found it!"
end
```

The `=~` operator returns the index where the match starts (6 in this case) or `nil` if there is no match. Since any integer is truthy in Ruby, this works naturally in conditionals.

The `match` method returns a `MatchData` object with detailed information about the match:

```ruby
match = text.match(/World/)
puts match[0]  # => "World"
```

For simple boolean checks, `match?` is the best choice because it avoids creating a `MatchData` object:

```ruby
# Using match? (returns boolean, faster)
text.match?(/World/)  # => true
```

Use `match?` when you only need to know whether a pattern matches, and `match` when you need to extract data from the match.

### Common Patterns

Ruby regex supports standard character classes, quantifiers, and anchors. Here is a reference of the most commonly used patterns:

```ruby
# Character classes
/[aeiou]/     # Any vowel
/[^aeiou]/    # Any non-vowel
/[a-z]/       # Lowercase letter
/\d/          # Digit (same as [0-9])
/\w/          # Word char (same as [a-zA-Z0-9_])
/\s/          # Whitespace

# Quantifiers
/a*/          # Zero or more
/a+/          # One or more
/a?/          # Zero or one
/a{3}/        # Exactly 3
/a{2,4}/      # Between 2 and 4

# Anchors
/^start/      # Beginning of line
/end$/        # End of line
/\bword\b/    # Word boundary
```

Character classes define what characters can appear at a given position. Quantifiers define how many times a pattern can repeat. Anchors pin the match to a position in the string without consuming characters.

### Modifiers

Flags after the closing slash change how the pattern is interpreted:

```ruby
/pattern/i    # Case insensitive
/pattern/m    # Multiline (. matches newlines)
/pattern/x    # Extended (allows whitespace/comments)
```

The `x` modifier is especially useful for complex patterns because it lets you add whitespace and comments for readability.

## Common Pitfalls
1. **Using match instead of match? for boolean checks** — `match` allocates a `MatchData` object every time, which is wasteful when you only need true/false. Use `match?` for conditional checks.
2. **Forgetting anchors** — `/\d+/` matches "abc123def" because it finds digits anywhere in the string. If you want the entire string to be digits, use `/\A\d+\z/` with start-of-string and end-of-string anchors.

## Best Practices
1. **Use match? for validation, match for extraction** — Choose the right method based on whether you need the match details or just a boolean result.
2. **Use the x modifier for complex patterns** — Breaking a long regex across multiple lines with comments makes it far easier to maintain and review.

## Summary
- Ruby supports regex literals with `/pattern/` syntax and three main matching methods: `=~`, `match`, and `match?`.
- Character classes, quantifiers, and anchors form the building blocks of regex patterns.
- Use `match?` for boolean checks and `match` when you need to extract matched data.

## Code Examples

**Demonstrates using match? for fast validation and match with named captures for extracting email components.**

```ruby
# Email validation with regex
email = "user@example.com"

# Quick boolean check (fastest)
puts email.match?(/\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i)
# => true

# Extracting parts with match
if m = email.match(/\A(?<local>[\w+\-.]+)@(?<domain>[a-z\d\-.]+\.[a-z]+)\z/i)
  puts m[:local]   # => user
  puts m[:domain]  # => example.com
end
```


## Resources

- [Regexp Class](https://docs.ruby-lang.org/en/4.0/Regexp.html) — Official Regexp documentation

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*