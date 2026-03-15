---
source_course: "ruby"
source_lesson: "ruby-string-methods"
---

# String Methods & Ruby 4.0 Improvements

## Introduction
Ruby's `String` class has one of the richest method sets of any language. From simple search-and-replace to regex-powered scanning, these methods handle the text processing tasks that developers face daily. Ruby 4.0 adds targeted improvements like `strip` with character selectors.

## Key Concepts
- **sub vs gsub**: `sub` replaces the first occurrence of a pattern; `gsub` replaces all occurrences. Both accept strings or regex.
- **scan**: Returns an array of all matches for a pattern in a string, optionally with capture groups.
- **split**: Breaks a string into an array using a delimiter (string, regex, or whitespace by default).
- **strip with selectors (Ruby 4.0)**: The `strip`, `lstrip`, and `rstrip` methods now accept a character argument specifying what to remove.

## Real World Context
Processing CSV exports, cleaning user input, and transforming data between formats are everyday tasks. A developer importing product data might need to strip leading zeros from SKU codes, convert snake_case field names to camelCase, and extract version numbers from description text — all in the same pipeline. Ruby's string methods handle each of these concisely.

## Deep Dive

### Search & Replace

The `sub` and `gsub` methods are your primary tools for string replacement. `sub` replaces only the first match, while `gsub` replaces all matches:

```ruby
text = "Hello World, Hello Ruby"

# Replace first occurrence
text.sub("Hello", "Hi")     # => "Hi World, Hello Ruby"

# Replace all occurrences
text.gsub("Hello", "Hi")    # => "Hi World, Hi Ruby"
```

Both methods also accept regex patterns and blocks. The block form is powerful for dynamic replacements:

```ruby
# With regex and captures
"john_doe".gsub(/_(.)/){ $1.upcase }  # => "johnDoe"
```

The block receives each match and the global variable `$1` holds the first capture group. This pattern is commonly used for case conversion.

### Scanning All Matches

The `scan` method returns an array of all matches. With capture groups, it returns an array of arrays:

```ruby
"ruby 2.7, ruby 3.0, ruby 4.0".scan(/ruby (\d+\.\d+)/)
# => [["2.7"], ["3.0"], ["4.0"]]

# Just the versions (no capture group)
"ruby 2.7, ruby 3.0, ruby 4.0".scan(/\d+\.\d+/)
# => ["2.7", "3.0", "4.0"]
```

When the regex contains capture groups, `scan` returns only the captured portions. Without capture groups, it returns the full matches. This distinction matters when designing your pattern.

### Ruby 4.0: String#strip with Selectors

Ruby 4.0 enhances `strip`, `lstrip`, and `rstrip` to accept character selectors, making it easy to remove specific characters from string edges:

```ruby
# Traditional - removes whitespace
"  hello  ".strip  # => "hello"

# Ruby 4.0 - specify what to strip
"###hello###".delete("#")     # => "hello"
"...hello...".lstrip(".")    # => "hello..."
"000123".lstrip("0")          # => "123"

# Multiple characters
"<>hello<>".strip("<>")      # => "hello"
```

Previously, removing specific leading or trailing characters required regex or manual slicing. The new selector syntax is cleaner and more expressive.

### Splitting

The `split` method breaks a string into an array. It accepts a string delimiter, a regex, or defaults to splitting on whitespace:

```ruby
"a,b,c".split(",")           # => ["a", "b", "c"]
"a  b  c".split              # => ["a", "b", "c"] (splits on whitespace)
"a1b2c3".split(/\d/)         # => ["a", "b", "c"]
```

When splitting on whitespace (no argument), `split` also strips leading whitespace and collapses multiple spaces, which is usually what you want for text processing.

## Common Pitfalls
1. **Confusing sub and gsub** — Using `sub` when you intend to replace all occurrences leaves subsequent matches untouched. Always choose `gsub` when you want a global replacement.
2. **Forgetting that scan with captures returns nested arrays** — `"ab12cd34".scan(/(\d+)/)` returns `[["12"], ["34"]]`, not `["12", "34"]`. Remove the capture group or use `.flatten` if you want a flat array.

## Best Practices
1. **Use scan for extraction, gsub for transformation** — When you need to collect all matches, use `scan`. When you need to transform a string, use `gsub` with a block.
2. **Prefer strip with selectors over regex for edge trimming (Ruby 4.0)** — The new `strip("#")` syntax is more readable than `gsub(/\A#+|#+\z/, "")` and clearly communicates intent.

## Summary
- `sub` replaces the first match; `gsub` replaces all matches. Both accept strings, regex, and blocks.
- `scan` extracts all matches into an array, with behavior that changes based on whether capture groups are present.
- Ruby 4.0's `strip` with character selectors simplifies removing specific characters from string edges.

## Code Examples

**Demonstrates a practical text processing pipeline combining strip, scan with captures, and sub for data cleaning.**

```ruby
# Text processing pipeline: clean and transform data
raw_input = "  ##SKU: 00042, Name: Widget Pro##  "

# Step 1: Strip whitespace, then custom characters
cleaned = raw_input.strip.delete("#").strip
# => "SKU: 00042, Name: Widget Pro"

# Step 2: Extract fields with scan
fields = cleaned.scan(/(\w+): ([^,]+)/)
# => [["SKU", "00042"], ["Name", "Widget Pro"]]

# Step 3: Build a hash and clean the SKU
data = fields.to_h
data["SKU"] = data["SKU"].sub(/\A0+/, "")
puts data
# => {"SKU"=>"42", "Name"=>"Widget Pro"}
```


## Resources

- [String Class](https://docs.ruby-lang.org/en/4.0/String.html) — Official Ruby documentation for the String class, including all manipulation methods.

---

> 📘 *This lesson is part of the [Ruby Language Foundations](https://stanza.dev/courses/ruby) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*