---
source_course: "javascript"
source_lesson: "javascript-error-handling-patterns"
---

# Error Handling Patterns

## Introduction

Beyond basic try-catch, JavaScript offers several patterns for handling errors elegantly. From async error handling to defensive programming, these patterns make your code more robust and maintainable.

## Key Concepts

**Guard Clause**: Early return that handles edge cases first.

**Fail Fast**: Detect and report errors as early as possible.

**Error Boundary**: A component/function that catches errors from its children.

## Real World Context

Production applications need multiple layers of error handling: input validation, operational error handling, and global error logging. These patterns help organize error handling at each level.

## Deep Dive

### Guard Clauses (Fail Fast)

```javascript
// Instead of deeply nested conditionals
function processOrder(order) {
  if (order) {
    if (order.items) {
      if (order.items.length > 0) {
        // Process order
      }
    }
  }
}

// Use guard clauses
function processOrder(order) {
  if (!order) throw new Error('Order required');
  if (!order.items) throw new Error('Order items required');
  if (order.items.length === 0) throw new Error('Order cannot be empty');
  
  // Happy path - process order
}
```

### Result Objects Pattern

```javascript
// Return success/error object instead of throwing
function parseConfig(json) {
  try {
    const config = JSON.parse(json);
    return { success: true, data: config };
  } catch (error) {
    return { success: false, error: error.message };
  }
}

const result = parseConfig(input);
if (result.success) {
  useConfig(result.data);
} else {
  showError(result.error);
}
```

### Global Error Handler

```javascript
// Browser: catch unhandled errors
window.addEventListener('error', (event) => {
  console.error('Unhandled error:', event.error);
  logToServer(event.error);
});

// Browser: catch unhandled promise rejections
window.addEventListener('unhandledrejection', (event) => {
  console.error('Unhandled rejection:', event.reason);
  logToServer(event.reason);
});

// Node.js
process.on('uncaughtException', (error) => {
  console.error('Uncaught:', error);
  process.exit(1);
});

process.on('unhandledRejection', (reason) => {
  console.error('Unhandled rejection:', reason);
});
```

### Wrapper Functions

```javascript
// Safe JSON parse
function safeJsonParse(json, fallback = null) {
  try {
    return JSON.parse(json);
  } catch {
    return fallback;
  }
}

const data = safeJsonParse(maybeInvalidJson, {});

// Safe function execution
function tryCatch(fn, fallback = undefined) {
  try {
    return fn();
  } catch {
    return fallback;
  }
}

const result = tryCatch(() => riskyOperation(), 'default');
```

### Assertion Functions

```javascript
function assert(condition, message) {
  if (!condition) {
    throw new Error(message || 'Assertion failed');
  }
}

function assertType(value, type, name) {
  if (typeof value !== type) {
    throw new TypeError(
      `Expected ${name} to be ${type}, got ${typeof value}`
    );
  }
}

// Usage
function divide(a, b) {
  assertType(a, 'number', 'a');
  assertType(b, 'number', 'b');
  assert(b !== 0, 'Cannot divide by zero');
  return a / b;
}
```

## Common Pitfalls

1. **Swallowing errors in wrappers**: Always log or handle appropriately.
2. **Not distinguishing error types**: Operational vs programming errors need different handling.
3. **Overusing try-catch**: Use for exceptional cases, not normal control flow.

## Best Practices

- **Fail fast**: Validate inputs early and throw immediately.
- **Use guard clauses**: Reduce nesting, improve readability.
- **Set up global handlers**: Catch unhandled errors in production.
- **Log all errors**: Even when providing fallback values.

## Summary

Guard clauses validate early and reduce nesting. Result objects avoid exceptions for expected failures. Global handlers catch unhandled errors. Wrapper functions provide safe defaults. Assertion functions enforce contracts. Layer these patterns for robust error handling.

## Code Examples

**Guard Clauses (Fail Fast)**

```javascript
// Instead of deeply nested conditionals
function processOrder(order) {
  if (order) {
    if (order.items) {
      if (order.items.length > 0) {
        // Process order
      }
    }
  }
}

// Use guard clauses
function processOrder(order) {
  if (!order) throw new Error('Order required');
  if (!order.items) throw new Error('Order items required');
  if (order.items.length === 0) throw new Error('Order cannot be empty');
  
  // Happy path - process order
}
```

**Result Objects Pattern**

```javascript
// Return success/error object instead of throwing
function parseConfig(json) {
  try {
    const config = JSON.parse(json);
    return { success: true, data: config };
  } catch (error) {
    return { success: false, error: error.message };
  }
}

const result = parseConfig(input);
if (result.success) {
  useConfig(result.data);
} else {
  showError(result.error);
}
```


## Resources

- [MDN: Control Flow and Error Handling](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Control_flow_and_error_handling) — Error handling guide

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*