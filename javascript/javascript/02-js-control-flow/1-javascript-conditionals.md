---
source_course: "javascript"
source_lesson: "javascript-conditionals"
---

# Conditionals: if, else, switch

## Introduction

Programs need to make decisions. Should we show an error message? Is the user logged in? Conditionals are the branching points that let your code take different paths based on conditions. Mastering them is essential for writing logic that responds intelligently to different situations.

## Key Concepts

**Conditional Statement**: Code that executes different branches based on whether a condition is true or false.

**Truthy/Falsy**: Values that evaluate to true or false in a boolean context (covered in previous lesson).

**Short-circuit Evaluation**: Logical operators that stop evaluating as soon as the result is determined.

## Real World Context

Every application uses conditionals: showing different UI for logged-in vs guest users, validating form inputs, handling API response states, implementing feature flags, and routing logic. Clean conditional code is the backbone of maintainable applications.

## Deep Dive

### if...else Statements

```javascript
const score = 85;

if (score >= 90) {
  console.log('A');
} else if (score >= 80) {
  console.log('B');
} else if (score >= 70) {
  console.log('C');
} else {
  console.log('F');
}
// Output: 'B'
```

### Ternary Operator

For simple conditions, use the ternary operator:

```javascript
const status = age >= 18 ? 'adult' : 'minor';

// Nested ternaries (use sparingly)
const grade = score >= 90 ? 'A' : score >= 80 ? 'B' : 'C';
```

### switch Statement

```javascript
const day = 'Monday';

switch (day) {
  case 'Monday':
  case 'Tuesday':
  case 'Wednesday':
  case 'Thursday':
  case 'Friday':
    console.log('Weekday');
    break;
  case 'Saturday':
  case 'Sunday':
    console.log('Weekend');
    break;
  default:
    console.log('Invalid day');
}
```

### Logical Operators for Conditionals

```javascript
// AND - both must be true
if (isLoggedIn && hasPermission) {
  showDashboard();
}

// OR - at least one must be true
if (isAdmin || isOwner) {
  showEditButton();
}

// Short-circuit for conditional execution
isLoggedIn && showWelcome();  // Only calls showWelcome if logged in
user || redirectToLogin();    // Redirects if user is falsy
```

### Optional Chaining in Conditions

```javascript
// Safe property access
if (user?.profile?.settings?.darkMode) {
  enableDarkMode();
}
```

## Common Pitfalls

1. **Assignment instead of comparison**: `if (x = 5)` assigns, `if (x === 5)` compares.
2. **Forgetting `break` in switch**: Causes fallthrough to next case.
3. **Over-nested conditionals**: Deeply nested if/else is hard to read—consider early returns.

## Best Practices

- **Use early returns**: Reduce nesting by handling edge cases first.
- **Prefer `===` over `==`**: Avoid type coercion bugs.
- **Use object lookups for many conditions**: Instead of long switch statements.
- **Keep ternaries simple**: Nest at most one level.

## Summary

Conditionals (`if`, `else if`, `else`, `switch`) control program flow based on conditions. The ternary operator provides concise single-expression conditionals. Use logical operators (`&&`, `||`) for compound conditions and short-circuit evaluation. Always use strict equality (`===`) and prefer early returns over deep nesting.

## Code Examples

**if...else Statements**

```javascript
const score = 85;

if (score >= 90) {
  console.log('A');
} else if (score >= 80) {
  console.log('B');
} else if (score >= 70) {
  console.log('C');
} else {
  console.log('F');
}
// Output: 'B'
```

**Ternary Operator**

```javascript
const status = age >= 18 ? 'adult' : 'minor';

// Nested ternaries (use sparingly)
const grade = score >= 90 ? 'A' : score >= 80 ? 'B' : 'C';
```


## Resources

- [MDN: if...else](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/if...else) — Complete guide to conditional statements
- [MDN: switch](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/switch) — Switch statement reference

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*