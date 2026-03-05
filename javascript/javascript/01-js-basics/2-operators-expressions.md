---
source_course: "javascript"
source_lesson: "javascript-operators-expressions"
---

# Operators & Expressions

## Introduction

Operators are the verbs of programming—they perform actions on values. From basic arithmetic to complex logical comparisons, mastering operators is essential for writing expressive, efficient code. JavaScript's operators include some unique behaviors that can surprise developers from other languages.

## Key Concepts

**Operator**: A symbol that performs an operation on one or more operands (values).

**Expression**: A combination of values, variables, and operators that evaluates to a single value.

**Operand**: The value(s) that an operator acts upon.

## Real World Context

In real applications, you'll use operators constantly: calculating totals in shopping carts, validating form inputs with logical operators, comparing user permissions, and short-circuiting expensive operations. Understanding operator precedence prevents subtle bugs in complex conditions.

## Deep Dive

### Arithmetic Operators

```javascript
const a = 10, b = 3;
a + b;   // 13 (addition)
a - b;   // 7 (subtraction)
a * b;   // 30 (multiplication)
a / b;   // 3.333... (division)
a % b;   // 1 (modulo/remainder)
a ** b;  // 1000 (exponentiation, ES2016)
```

### Comparison Operators

```javascript
5 == '5';   // true (loose equality, type coercion)
5 === '5';  // false (strict equality, no coercion)
5 != '5';   // false
5 !== '5';  // true
5 > 3;      // true
5 >= 5;     // true
```

### Logical Operators

```javascript
true && false;  // false (AND)
true || false;  // true (OR)
!true;          // false (NOT)

// Short-circuit evaluation
const user = null;
const name = user && user.name;  // null (doesn't throw)
const fallback = user || 'Guest'; // 'Guest'
```

### Nullish Coalescing (ES2020)

```javascript
const value = null ?? 'default';  // 'default'
const zero = 0 ?? 'default';      // 0 (only null/undefined trigger fallback)
const empty = '' || 'default';    // 'default' (|| treats '' as falsy)
const empty2 = '' ?? 'default';   // '' (?? only checks null/undefined)
```

### Optional Chaining (ES2020)

```javascript
const user = { profile: { name: 'Alice' } };
user?.profile?.name;    // 'Alice'
user?.settings?.theme;  // undefined (no error)
```

## Common Pitfalls

1. **Using `==` instead of `===`**: Loose equality performs type coercion leading to surprises like `0 == ''` being `true`.
2. **Confusing `||` with `??`**: The `||` operator treats `0`, `''`, and `false` as falsy, while `??` only checks for `null`/`undefined`.
3. **Operator precedence confusion**: `a && b || c` may not behave as expected—use parentheses for clarity.

## Best Practices

- **Always use `===` and `!==`**: Avoid implicit type coercion.
- **Use `??` for default values**: When `0` or `''` are valid values.
- **Use optional chaining (`?.`)**: Prevents "cannot read property of undefined" errors.
- **Parenthesize complex expressions**: Makes precedence explicit and code readable.

## Summary

JavaScript operators include arithmetic (`+`, `-`, `*`, `/`, `%`, `**`), comparison (`===`, `!==`, `>`, `<`), logical (`&&`, `||`, `!`), and modern additions like nullish coalescing (`??`) and optional chaining (`?.`). Always prefer strict equality and use modern operators for safer, more expressive code.

## Code Examples

**Nullish coalescing (??) vs logical OR (||)**

```javascript
const value = null ?? 'default';  // 'default'
const zero = 0 ?? 'default';      // 0 (only null/undefined trigger fallback)
const empty = '' || 'default';    // 'default' (|| treats '' as falsy)
const empty2 = '' ?? 'default';   // '' (?? only checks null/undefined)
```

**Optional chaining for safe property access**

```javascript
const user = { profile: { name: 'Alice' } };
user?.profile?.name;    // 'Alice'
user?.settings?.theme;  // undefined (no error)
```


## Resources

- [MDN: Expressions and Operators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_operators) — Complete guide to JavaScript operators
- [MDN: Nullish Coalescing Operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing) — Understanding the ?? operator

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*