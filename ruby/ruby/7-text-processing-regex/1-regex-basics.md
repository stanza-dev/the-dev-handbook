---
source_course: "ruby"
source_lesson: "ruby-regex-basics"
---

# Regular Expressions

Ruby has first-class support for Regex using the `/pattern/` syntax.

## Basic Matching

```ruby
text = "Hello World"

# Using =~
if text =~ /World/
  puts "Found it!"
end

# Using match
match = text.match(/World/)
puts match[0]  # => "World"

# Using match? (returns boolean, faster)
text.match?(/World/)  # => true
```

## Common Patterns

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

## Modifiers

```ruby
/pattern/i    # Case insensitive
/pattern/m    # Multiline (. matches newlines)
/pattern/x    # Extended (allows whitespace/comments)
```

## Resources

- [Regexp Class](https://docs.ruby-lang.org/en/4.0/Regexp.html) — Official Regexp documentation

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*