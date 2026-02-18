---
source_course: "javascript"
source_lesson: "javascript-regex-methods"
---

# String Methods with Regex

## Introduction

JavaScript strings have several methods that accept regex patterns. Knowing which method to use—and how regex flags affect them—is key to effective text processing.

## Key Concepts

**match()**: Find matches in a string.

**replace()/replaceAll()**: Replace matched text.

**split()**: Split string by pattern.

**search()**: Find index of first match.

## Deep Dive

### match()

```javascript
const str = 'The rain in Spain';

// Without global flag - returns first match with details
str.match(/ain/);
// ['ain', index: 5, input: '...', groups: undefined]

// With global flag - returns all matches
str.match(/ain/g);
// ['ain', 'ain', 'ain']

// No match returns null
str.match(/xyz/);  // null

// Safe pattern: use optional chaining
const matches = str.match(/xyz/g) ?? [];
```

### replace() and replaceAll()

```javascript
// replace() - first occurrence only (without g flag)
'hello hello'.replace('hello', 'hi');   // 'hi hello'
'hello hello'.replace(/hello/, 'hi');   // 'hi hello'

// replace() with global flag - all occurrences
'hello hello'.replace(/hello/g, 'hi');  // 'hi hi'

// replaceAll() - all occurrences (ES2021)
'hello hello'.replaceAll('hello', 'hi'); // 'hi hi'
'hello hello'.replaceAll(/hello/g, 'hi'); // 'hi hi' (g flag required!)

// With callback function
'hello world'.replace(/\w+/g, (match) => match.toUpperCase());
// 'HELLO WORLD'

// Callback with groups
'John Smith'.replace(/(\w+) (\w+)/, (match, first, last) => {
  return `${last}, ${first}`;
});
// 'Smith, John'
```

### split()

```javascript
// Split by regex
'a1b2c3'.split(/\d/);     // ['a', 'b', 'c', '']

// Limit results
'a1b2c3'.split(/\d/, 2);  // ['a', 'b']

// Keep delimiters with capturing group
'a1b2c3'.split(/(\d)/);   // ['a', '1', 'b', '2', 'c', '3', '']

// Split on multiple delimiters
'a,b;c:d'.split(/[,;:]/); // ['a', 'b', 'c', 'd']
```

### search()

```javascript
// Returns index of first match (like indexOf for regex)
'hello world'.search(/world/);  // 6
'hello world'.search(/xyz/);    // -1

// Case insensitive search
'Hello World'.search(/world/i); // 6
```

### test() and exec() (RegExp methods)

```javascript
// test() - boolean check
/\d+/.test('abc123');  // true

// exec() - detailed match info
const regex = /\d+/g;
let match;
while ((match = regex.exec('a1b22c333')) !== null) {
  console.log(match[0], 'at index', match.index);
}
// '1' at index 1
// '22' at index 3
// '333' at index 6
```

## Common Pitfalls

1. **match() with global returns array without groups**: Use matchAll() for groups.
2. **replaceAll() requires global flag with regex**: Or use string pattern.
3. **exec() with global flag is stateful**: regex.lastIndex advances.

## Best Practices

- **Use matchAll() for extraction**: Gets all matches with groups.
- **Use replaceAll() for simple replacements**: Clearer than `/g` flag.
- **Check for null from match()**: Returns null if no match.
- **Prefer test() for validation**: Faster than match() when you just need boolean.

## Summary

`match()` finds matches, `replace()`/`replaceAll()` substitutes text, `split()` divides strings, `search()` finds index. Use `g` flag for global operations. `matchAll()` is best for extraction with groups. Always handle null returns from `match()`.

## Resources

- [MDN: String.prototype.match()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/match) — match() reference
- [MDN: String.prototype.replace()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/replace) — replace() reference

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*