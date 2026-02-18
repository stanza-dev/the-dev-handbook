---
source_course: "javascript"
source_lesson: "javascript-type-coercion"
---

# Type Coercion & Conversion

## Introduction

JavaScript is famous—or infamous—for its automatic type conversions. When you add a number to a string or compare values of different types, JavaScript silently converts them. Understanding these rules prevents countless debugging sessions and interview failures.

## Key Concepts

**Type Coercion**: Automatic (implicit) conversion of values from one type to another by the JavaScript engine.

**Type Conversion**: Explicit conversion using functions like `Number()`, `String()`, `Boolean()`.

**Truthy/Falsy**: Values that evaluate to `true` or `false` in boolean contexts.

## Real World Context

Form inputs are always strings, so you must convert them before calculations. API responses may have unexpected types. Conditional checks rely on truthy/falsy evaluation. Understanding coercion is essential for writing robust validation logic.

## Deep Dive

### Explicit Conversion

```javascript
// To Number
Number('42');       // 42
Number('42px');     // NaN
Number(true);       // 1
Number(null);       // 0
Number(undefined);  // NaN
parseInt('42px');   // 42 (parses until non-digit)
parseFloat('3.14'); // 3.14

// To String
String(42);         // '42'
String(null);       // 'null'
String(undefined);  // 'undefined'
(42).toString();    // '42'

// To Boolean
Boolean(1);         // true
Boolean(0);         // false
Boolean('');        // false
Boolean('hello');   // true
```

### Falsy Values (only 8!)

```javascript
false
0
-0
0n        // BigInt zero
''        // empty string
null
undefined
NaN
```

Everything else is **truthy**, including `[]`, `{}`, and `'false'`.

### Implicit Coercion Gotchas

```javascript
'5' + 3;        // '53' (string concatenation)
'5' - 3;        // 2 (numeric subtraction)
'5' * '2';      // 10 (both converted to numbers)
[] + [];        // '' (empty string)
[] + {};        // '[object Object]'
{} + [];        // 0 (block statement + array)
```

### The `==` Coercion Rules

```javascript
null == undefined;  // true (special case)
'0' == 0;           // true (string to number)
0 == false;         // true (boolean to number)
'0' == false;       // true (both to number)
[] == false;        // true (array to primitive)
```

## Common Pitfalls

1. **Truthy empty array**: `[]` is truthy but `[] == false` is true due to coercion rules.
2. **NaN comparisons**: `NaN === NaN` is `false`. Use `Number.isNaN()` instead.
3. **String concatenation with `+`**: `'Total: ' + 5 + 10` gives `'Total: 510'`, not `'Total: 15'`.

## Best Practices

- **Be explicit**: Use `Number()`, `String()`, `Boolean()` when converting.
- **Use `===` always**: Avoid implicit coercion in comparisons.
- **Use `Number.isNaN()`**: Not the global `isNaN()` which coerces first.
- **Parenthesize arithmetic in string templates**: `` `Total: ${5 + 10}` ``

## Summary

JavaScript has 8 falsy values; everything else is truthy. Use explicit conversion functions and strict equality to avoid coercion surprises. The `+` operator concatenates strings, while `-`, `*`, `/` convert to numbers. Understanding these rules is essential for writing predictable code.

## Resources

- [MDN: Type Coercion](https://developer.mozilla.org/en-US/docs/Glossary/Type_coercion) — Understanding implicit type conversion
- [MDN: Equality Comparisons](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness) — Deep dive into equality operators

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*