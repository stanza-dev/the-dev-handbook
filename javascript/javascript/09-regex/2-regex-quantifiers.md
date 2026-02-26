---
source_course: "javascript"
source_lesson: "javascript-regex-quantifiers"
---

# Quantifiers & Anchors

## Introduction

Quantifiers specify how many times a pattern should match. Anchors specify where in the string to look. Together, they let you build precise patterns for validation and extraction.

## Key Concepts

**Quantifier**: Specifies repetition (`+`, `*`, `?`, `{n}`).

**Anchor**: Matches a position, not a character (`^`, `$`, `\b`).

**Greedy vs Lazy**: How quantifiers consume characters.

## Deep Dive

### Quantifiers

```javascript
// ? - zero or one (optional)
/colou?r/.test('color');   // true
/colou?r/.test('colour');  // true

// * - zero or more
/go*gle/.test('ggle');     // true
/go*gle/.test('google');   // true
/go*gle/.test('gooooogle'); // true

// + - one or more
/go+gle/.test('ggle');     // false (needs at least one 'o')
/go+gle/.test('google');   // true

// {n} - exactly n times
/\d{3}/.test('123');       // true
/\d{3}/.test('12');        // false

// {n,m} - n to m times
/\d{2,4}/.test('123');     // true

// {n,} - n or more times
/\d{3,}/.test('12345');    // true
```

### Greedy vs Lazy

```javascript
const html = '<div>content</div>';

// Greedy (default) - matches as much as possible
html.match(/<.*>/);     // ['<div>content</div>'] (whole thing!)

// Lazy (add ?) - matches as little as possible  
html.match(/<.*?>/);    // ['<div>'] (just first tag)

// Lazy versions: *?, +?, ??, {n,m}?
```

### Anchors

```javascript
// ^ - start of string (or line with m flag)
/^hello/.test('hello world');  // true
/^hello/.test('say hello');    // false

// $ - end of string (or line with m flag)
/world$/.test('hello world');  // true
/world$/.test('world hello');  // false

// \b - word boundary
/\bcat\b/.test('the cat sat');  // true
/\bcat\b/.test('category');     // false

// \B - non-word boundary
/\Bcat\B/.test('category');     // false
/\Bcat/.test('concatenate');    // true
```

### Common Validation Patterns

```javascript
// Exact match
/^hello$/.test('hello');        // true
/^hello$/.test('hello world');  // false

// Phone number (US)
/^\d{3}-\d{3}-\d{4}$/.test('123-456-7890');  // true

// Alphanumeric only
/^[a-zA-Z0-9]+$/.test('Hello123');  // true

// No whitespace
/^\S+$/.test('nospaces');  // true
```

## Common Pitfalls

1. **Forgetting anchors**: `/\d{3}/` matches '123' in 'abc123xyz'.
2. **Greedy matching too much**: Use `?` for lazy matching.
3. **`^` in character class negates**: `[^abc]` means NOT a, b, or c.

## Best Practices

- **Use `^` and `$` for validation**: Ensure entire string matches.
- **Prefer lazy quantifiers for HTML/XML**: `.*?` between tags.
- **Use `\b` for word matching**: Prevents partial word matches.

## Summary

Quantifiers (`+`, `*`, `?`, `{n}`) specify repetition count. Anchors (`^`, `$`, `\b`) match positions. Add `?` after quantifiers for lazy (minimal) matching. Always use anchors for input validation to ensure the entire string matches.

## Code Examples

**Quantifiers**

```javascript
// ? - zero or one (optional)
/colou?r/.test('color');   // true
/colou?r/.test('colour');  // true

// * - zero or more
/go*gle/.test('ggle');     // true
/go*gle/.test('google');   // true
/go*gle/.test('gooooogle'); // true

// + - one or more
/go+gle/.test('ggle');     // false (needs at least one 'o')
/go+gle/.test('google');   // true

// {n} - exactly n times
/\d{3}/.test('123');       // true
/\d{3}/.test('12');        // false

// {n,m} - n to m times
```

**Greedy vs Lazy**

```javascript
const html = '<div>content</div>';

// Greedy (default) - matches as much as possible
html.match(/<.*>/);     // ['<div>content</div>'] (whole thing!)

// Lazy (add ?) - matches as little as possible  
html.match(/<.*?>/);    // ['<div>'] (just first tag)

// Lazy versions: *?, +?, ??, {n,m}?
```


## Resources

- [MDN: Quantifiers](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_expressions/Quantifiers) — Quantifier reference

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*