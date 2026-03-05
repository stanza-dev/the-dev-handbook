---
source_course: "javascript"
source_lesson: "javascript-throwing-errors"
---

# Throwing & Creating Errors

## Introduction

Sometimes your code needs to signal that something went wrong. The `throw` statement lets you create custom errors with meaningful messages. Understanding error types and custom errors makes debugging easier and APIs more predictable.

## Key Concepts

**throw**: Statement that generates an exception.

**Error Types**: Built-in error classes for different error categories.

**Custom Errors**: User-defined error classes for domain-specific errors.

## Real World Context

Input validation, business rule enforcement, API contract violations—all require throwing appropriate errors. Libraries use custom errors to help consumers understand what went wrong.

## Deep Dive

### The throw Statement

```javascript
// Throw an Error object
throw new Error('Something went wrong');

// You can throw anything (but don't)
throw 'error string';  // Works but loses stack trace
throw 42;              // Works but meaningless
throw { code: 500 };   // Works but loses Error features

// Always throw Error objects!
```

### Built-in Error Types

```javascript
// TypeError - wrong type
const obj = null;
obj.method();  // TypeError: Cannot read properties of null

// ReferenceError - undefined variable
console.log(undefinedVar);  // ReferenceError

// SyntaxError - invalid syntax (usually at parse time)
JSON.parse('invalid');  // SyntaxError: Unexpected token

// RangeError - value out of range
new Array(-1);  // RangeError: Invalid array length

// Throwing specific types
throw new TypeError('Expected a string');
throw new RangeError('Value must be between 0 and 100');
```

### Custom Error Classes

```javascript
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = 'ValidationError';
    this.field = field;
  }
}

class HttpError extends Error {
  constructor(status, message) {
    super(message);
    this.name = 'HttpError';
    this.status = status;
  }
}

// Usage
function validateEmail(email) {
  if (!email.includes('@')) {
    throw new ValidationError('Invalid email format', 'email');
  }
}

try {
  validateEmail('invalid');
} catch (error) {
  if (error instanceof ValidationError) {
    console.log(`Field: ${error.field}, Message: ${error.message}`);
  }
}
```

### Error Checking Patterns

```javascript
// Type checking with instanceof
try {
  riskyOperation();
} catch (error) {
  if (error instanceof TypeError) {
    // Handle type error
  } else if (error instanceof ValidationError) {
    // Handle validation error
  } else {
    throw error;  // Re-throw unknown errors
  }
}

// Check error.name
if (error.name === 'SyntaxError') {
  // Handle JSON parse errors etc.
}
```

### Cause Property (ES2022)

```javascript
try {
  try {
    JSON.parse(invalidJson);
  } catch (parseError) {
    throw new Error('Config loading failed', {
      cause: parseError
    });
  }
} catch (error) {
  console.log(error.message);      // 'Config loading failed'
  console.log(error.cause.message); // Original parse error
}
```

## Common Pitfalls

1. **Throwing strings**: Loses stack trace and error properties.
2. **Not setting name in custom errors**: Makes debugging harder.
3. **Forgetting to call super()**: Custom error won't behave correctly.

## Best Practices

- **Always throw Error objects**: Not strings or plain objects.
- **Use specific error types**: `TypeError`, `RangeError`, or custom classes.
- **Include context in messages**: What failed and why.
- **Use `cause` for error chaining**: Preserve original error information.

## Summary

Use `throw` to signal errors. Always throw Error objects, not strings. Use built-in error types (`TypeError`, `RangeError`, `SyntaxError`) when appropriate. Create custom error classes for domain-specific errors. Use the `cause` property to chain errors.

## Code Examples

**The throw Statement**

```javascript
// Throw an Error object
throw new Error('Something went wrong');

// You can throw anything (but don't)
throw 'error string';  // Works but loses stack trace
throw 42;              // Works but meaningless
throw { code: 500 };   // Works but loses Error features

// Always throw Error objects!
```

**Built-in Error Types**

```javascript
// TypeError - wrong type
const obj = null;
obj.method();  // TypeError: Cannot read properties of null

// ReferenceError - undefined variable
console.log(undefinedVar);  // ReferenceError

// SyntaxError - invalid syntax (usually at parse time)
JSON.parse('invalid');  // SyntaxError: Unexpected token

// RangeError - value out of range
new Array(-1);  // RangeError: Invalid array length

// Throwing specific types
throw new TypeError('Expected a string');
throw new RangeError('Value must be between 0 and 100');
```


## Resources

- [MDN: throw](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/throw) — throw statement reference
- [MDN: Error Types](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error#error_types) — Built-in error types

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*