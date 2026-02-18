---
source_course: "javascript"
source_lesson: "javascript-regex-basics"
---

# Regular Expression Fundamentals

## Introduction

Regular expressions (regex) are patterns for matching text. They're powerful but cryptic—a double-edged sword that can validate emails in one line or create maintenance nightmares. Understanding the basics unlocks powerful text processing capabilities.

## Key Concepts

**Regular Expression**: A pattern that describes a set of strings.

**Literal**: Characters that match themselves (like `hello`).

**Metacharacters**: Special characters with meaning (like `.`, `*`, `+`).

## Real World Context

Form validation, search and replace, parsing logs, extracting data from strings—regex appears throughout web development. Most validation libraries use regex under the hood.

## Deep Dive

### Creating Regular Expressions

```javascript
// Literal syntax (preferred for static patterns)
const regex1 = /hello/;

// Constructor syntax (for dynamic patterns)
const pattern = 'hello';
const regex2 = new RegExp(pattern);

// With flags
const regex3 = /hello/gi;  // g=global, i=case-insensitive
const regex4 = new RegExp('hello', 'gi');
```

### Testing and Matching

```javascript
const regex = /world/i;

// test() - returns boolean
regex.test('Hello World');  // true

// match() - returns matches
'Hello World'.match(regex);  // ['World']
'Hello World World'.match(/world/gi);  // ['World', 'World']

// exec() - detailed info (use in loop for global)
regex.exec('Hello World');  // ['World', index: 6, input: '...']
```

### Basic Patterns

```javascript
// Literals
/hello/.test('hello world');  // true

// Character classes
/[aeiou]/.test('hello');  // true (matches any vowel)
/[0-9]/.test('abc123');   // true
/[^0-9]/.test('abc123');  // true (^ negates in character class)

// Predefined classes
/\d/.test('abc123');  // true (digit)
/\w/.test('hello');   // true (word char: a-z, A-Z, 0-9, _)
/\s/.test('hello world');  // true (whitespace)
/\D/.test('abc');     // true (non-digit, uppercase negates)

// Any character (except newline)
/./.test('x');  // true
```

### Flags

```javascript
// g - global (find all matches, not just first)
'hello hello'.match(/hello/);   // ['hello']
'hello hello'.match(/hello/g);  // ['hello', 'hello']

// i - case insensitive
/hello/i.test('HELLO');  // true

// m - multiline (^ and $ match line boundaries)
/^hello/m.test('world\nhello');  // true

// s - dotAll (. matches newlines too)
/hello.world/s.test('hello\nworld');  // true

// u - unicode (proper unicode handling)
/\p{L}/u.test('π');  // true (any letter)

// d - indices (ES2022, capture group indices)
const result = /(?<name>\w+)/.exec('hello');
```

## Common Pitfalls

1. **Escaping special characters**: `.` matches any char; `\.` matches literal dot.
2. **Greedy by default**: `.*` matches as much as possible.
3. **Global flag changes exec()**: Must loop; index advances between calls.

## Best Practices

- **Use literal syntax when possible**: `/pattern/` is cleaner than `new RegExp()`.
- **Use named groups for clarity**: `(?<year>\d{4})` instead of `(\d{4})`.
- **Escape user input**: `RegExp.escape()` or manual escaping for dynamic patterns.
- **Comment complex regex**: Or use verbose mode with libraries.

## Summary

Regex patterns match text using literals and metacharacters. Use `/pattern/flags` syntax. `test()` returns boolean, `match()` returns matches. Flags modify behavior: `g` for global, `i` for case-insensitive, `m` for multiline. Always escape special characters in literal strings.

## Resources

- [MDN: Regular Expressions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_expressions) — Complete regex guide
- [MDN: RegExp](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp) — RegExp object reference

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*