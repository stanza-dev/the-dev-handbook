---
source_course: "javascript"
source_lesson: "javascript-strings-template-literals"
---

# Strings & Template Literals

## Introduction

Strings are everywhere in programming—user names, error messages, URLs, JSON data. JavaScript provides powerful string manipulation capabilities, and ES6's template literals revolutionized how we work with dynamic strings. Master these tools to write cleaner, more readable code.

## Key Concepts

**String**: A sequence of characters representing text. Immutable in JavaScript.

**Template Literal**: Strings delimited by backticks (`) that support interpolation and multi-line content.

**String Method**: Built-in functions for manipulating strings (they return new strings since strings are immutable).

## Real World Context

Building URLs with query parameters, formatting user messages, parsing CSV data, validating email formats—strings are involved in nearly every feature. Template literals eliminate messy concatenation, and string methods power search, filtering, and data transformation.

## Deep Dive

### String Creation

```javascript
const single = 'Hello';
const double = "World";
const template = `Hello, ${name}!`;
```

### Template Literals (ES6)

```javascript
const user = 'Alice';
const greeting = `Hello, ${user}!`; // 'Hello, Alice!'

// Multi-line strings
const html = `
  <div>
    <h1>${title}</h1>
    <p>${content}</p>
  </div>
`;

// Expression interpolation
const price = 19.99;
const message = `Total: $${(price * 1.1).toFixed(2)}`; // 'Total: $21.99'
```

### Essential String Methods

```javascript
const str = '  Hello, World!  ';

// Searching
str.includes('World');     // true
str.startsWith('  He');    // true
str.endsWith('!  ');       // true
str.indexOf('o');          // 4 (first occurrence)

// Extracting
str.slice(2, 7);           // 'Hello'
str.substring(2, 7);       // 'Hello'
str.charAt(2);             // 'H'
str.at(-1);                // ' ' (ES2022, negative indexing)

// Transforming
str.trim();                // 'Hello, World!'
str.toLowerCase();         // '  hello, world!  '
str.toUpperCase();         // '  HELLO, WORLD!  '
str.replace('World', 'JS'); // '  Hello, JS!  '
str.replaceAll(' ', '-');  // '--Hello,-World!--'

// Splitting and joining
'a,b,c'.split(',');        // ['a', 'b', 'c']
['a', 'b', 'c'].join('-'); // 'a-b-c'

// Padding (ES2017)
'5'.padStart(3, '0');      // '005'
'5'.padEnd(3, '0');        // '500'
```

### Tagged Template Literals

```javascript
function highlight(strings, ...values) {
  return strings.reduce((acc, str, i) => 
    acc + str + (values[i] ? `<mark>${values[i]}</mark>` : ''), '');
}

const name = 'Alice';
highlight`Hello, ${name}!`; // 'Hello, <mark>Alice</mark>!'
```

## Common Pitfalls

1. **Forgetting strings are immutable**: `str.toUpperCase()` returns a NEW string; it doesn't modify `str`.
2. **Using `+` for complex concatenation**: Template literals are cleaner and less error-prone.
3. **Confusing `slice` and `substring`**: `slice` handles negative indices; `substring` swaps arguments if start > end.

## Best Practices

- **Use template literals**: For any string with variables or spanning multiple lines.
- **Use `includes()` over `indexOf()`**: More readable for existence checks.
- **Use `trim()` on user input**: Prevents issues with accidental whitespace.
- **Use `replaceAll()` for global replacement**: Or use regex with `g` flag.

## Summary

Strings in JavaScript are immutable sequences of characters. Template literals (backticks) enable interpolation with `${}` and multi-line strings. Key methods include `includes()`, `slice()`, `trim()`, `split()`, and `replace()`. Always remember that string methods return new strings.

## Code Examples

**Template literals with expression interpolation**

```javascript
const user = 'Alice';
const greeting = `Hello, ${user}!`; // 'Hello, Alice!'

const price = 19.99;
const message = `Total: $${(price * 1.1).toFixed(2)}`; // 'Total: $21.99'
```

**Essential string methods**

```javascript
const str = '  Hello, World!  ';
str.includes('World');     // true
str.startsWith('  He');    // true
str.trim();                // 'Hello, World!'
str.replace('World', 'JS'); // '  Hello, JS!  '
'a,b,c'.split(',');        // ['a', 'b', 'c']
'5'.padStart(3, '0');      // '005'
```


## Resources

- [MDN: String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String) — Complete String reference with all methods
- [MDN: Template Literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals) — Guide to template literals and tagged templates

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*