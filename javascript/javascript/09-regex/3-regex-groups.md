---
source_course: "javascript"
source_lesson: "javascript-regex-groups"
---

# Groups & Capturing

## Introduction

Groups let you extract parts of a match, apply quantifiers to multiple characters, and create backreferences. Named groups make patterns self-documenting. These features transform regex from pattern matching to powerful text extraction.

## Key Concepts

**Capturing Group**: `()` captures matched text for later use.

**Non-capturing Group**: `(?:)` groups without capturing.

**Named Group**: `(?<name>)` captures with a descriptive name.

## Deep Dive

### Basic Groups

```javascript
// Capturing groups
const match = 'John Smith'.match(/(\w+) (\w+)/);
match[0];  // 'John Smith' (full match)
match[1];  // 'John' (first group)
match[2];  // 'Smith' (second group)

// Group quantifiers
/(ab)+/.test('ababab');  // true (matches 'ab' one or more times)

// Alternation within groups
/(cat|dog)/.test('I have a cat');  // true
```

### Named Groups (ES2018)

```javascript
const dateRegex = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;
const match = '2024-12-25'.match(dateRegex);

match.groups.year;   // '2024'
match.groups.month;  // '12'
match.groups.day;    // '25'

// Destructuring
const { groups: { year, month, day } } = '2024-12-25'.match(dateRegex);
```

### Non-capturing Groups

```javascript
// When you need grouping but don't need the capture
/(?:ab)+/.test('ababab');  // true

// Compare:
'ababab'.match(/(ab)+/);   // ['ababab', 'ab'] (captures last 'ab')
'ababab'.match(/(?:ab)+/); // ['ababab'] (no extra capture)
```

### Backreferences

```javascript
// Reference earlier capture
/(\w+) and \1/.test('cat and cat');  // true
/(\w+) and \1/.test('cat and dog');  // false

// Named backreference
/(?<word>\w+) and \k<word>/.test('cat and cat');  // true

// Find duplicated words
/\b(\w+)\s+\1\b/.test('the the');  // true
```

### Using Groups in Replace

```javascript
// Numbered groups
'John Smith'.replace(/(\w+) (\w+)/, '$2, $1');
// 'Smith, John'

// Named groups
'2024-12-25'.replace(
  /(?<y>\d{4})-(?<m>\d{2})-(?<d>\d{2})/,
  '$<m>/$<d>/$<y>'
);
// '12/25/2024'

// With function
'hello world'.replace(/(\w+)/g, (match, p1) => p1.toUpperCase());
// 'HELLO WORLD'
```

### matchAll() for Multiple Matches

```javascript
const regex = /(\w+)@(\w+\.\w+)/g;
const text = 'Contact: alice@example.com, bob@test.org';

for (const match of text.matchAll(regex)) {
  console.log(match[0]);  // Full email
  console.log(match[1]);  // Username
  console.log(match[2]);  // Domain
}
```

## Common Pitfalls

1. **Too many captures slow matching**: Use `(?:)` when you don't need the value.
2. **Backreference numbering**: Groups are numbered left-to-right by opening paren.
3. **Global flag with match()**: Returns array of matches, not groups. Use `matchAll()`.

## Best Practices

- **Use named groups for readability**: `(?<year>\d{4})` is self-documenting.
- **Use non-capturing groups for alternation**: `(?:cat|dog)` when you don't need capture.
- **Use matchAll() for extraction**: Gives all matches with all groups.

## Summary

Capturing groups `()` extract matched text. Named groups `(?<name>)` add clarity. Non-capturing groups `(?:)` group without overhead. Backreferences (`\1`, `\k<name>`) match earlier captures. Use `$1` or `$<name>` in replacements.

## Resources

- [MDN: Groups and backreferences](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_expressions/Groups_and_backreferences) — Groups reference
- [MDN: String.prototype.matchAll()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/matchAll) — matchAll() reference

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*