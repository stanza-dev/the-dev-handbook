---
source_course: "javascript-async-patterns"
source_lesson: "javascript-async-patterns-error-handling-async"
---

# Async Error Handling Patterns

## Introduction

Async errors are trickier than sync errors—they can slip through try/catch, cause unhandled rejections, and be hard to trace. Mastering async error handling patterns is essential for building reliable applications.

## Key Concepts

**Unhandled Rejection**: A rejected promise without a .catch() or try/catch.

**Error Propagation**: How errors flow through promise chains and async functions.

**Result Pattern**: Returning {ok, value/error} instead of throwing.

## Deep Dive

### Try/Catch with Async/Await

```javascript
async function fetchUser(id) {
  try {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    return await response.json();
  } catch (error) {
    // Handles fetch errors AND thrown errors
    console.error('Failed to fetch user:', error);
    throw error;  // Re-throw for caller to handle
  }
}
```

### Handling Multiple Awaits

```javascript
// Each await needs consideration
async function processOrder(orderId) {
  try {
    const order = await fetchOrder(orderId);
    const user = await fetchUser(order.userId);
    const result = await chargeUser(user, order.total);
    return result;
  } catch (error) {
    // Which operation failed? Hard to tell!
    throw error;
  }
}

// Better: specific error handling
async function processOrderBetter(orderId) {
  let order, user;
  
  try {
    order = await fetchOrder(orderId);
  } catch (e) {
    throw new Error(`Failed to fetch order: ${e.message}`);
  }
  
  try {
    user = await fetchUser(order.userId);
  } catch (e) {
    throw new Error(`Failed to fetch user: ${e.message}`);
  }
  
  try {
    return await chargeUser(user, order.total);
  } catch (e) {
    throw new Error(`Failed to charge: ${e.message}`);
  }
}
```

### Global Unhandled Rejection Handler

```javascript
// Browser
window.addEventListener('unhandledrejection', (event) => {
  console.error('Unhandled rejection:', event.reason);
  // Prevent default browser behavior (console error)
  event.preventDefault();
  // Log to service
  logError(event.reason);
});

// Node.js
process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled Rejection:', reason);
});
```

### Result Pattern (No-Throw)

```javascript
// Return result object instead of throwing
async function safeAsync(fn) {
  try {
    const data = await fn();
    return { ok: true, data };
  } catch (error) {
    return { ok: false, error };
  }
}

// Usage
const result = await safeAsync(() => fetch('/api/data'));
if (result.ok) {
  console.log(result.data);
} else {
  console.error(result.error);
}
```

### Promise.allSettled for Partial Failures

```javascript
async function fetchAllUsers(ids) {
  const results = await Promise.allSettled(
    ids.map(id => fetchUser(id))
  );
  
  const users = [];
  const errors = [];
  
  results.forEach((result, i) => {
    if (result.status === 'fulfilled') {
      users.push(result.value);
    } else {
      errors.push({ id: ids[i], error: result.reason });
    }
  });
  
  return { users, errors };
}
```

### Retry Pattern

```javascript
async function retry(fn, attempts = 3, delay = 1000) {
  for (let i = 0; i < attempts; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === attempts - 1) throw error;
      console.log(`Attempt ${i + 1} failed, retrying...`);
      await new Promise(r => setTimeout(r, delay * (i + 1)));
    }
  }
}

// Usage
const data = await retry(() => fetch('/flaky-api'), 3, 1000);
```

## Common Pitfalls

1. **Swallowing errors**: Catching without re-throwing or logging.
2. **Missing await**: Promise rejection becomes unhandled.
3. **catch() returning undefined**: Downstream code gets undefined.

## Best Practices

- **Always handle or re-throw errors**: Never silently swallow.
- **Use global handlers as safety net**: Not as primary error handling.
- **Add context to errors**: "Failed to fetch user" not just "Network error".
- **Consider Result pattern**: For operations that commonly fail.

## Summary

Use try/catch with async/await. Handle errors specifically when possible. Set up global handlers for safety. Consider Result pattern for expected failures. Use Promise.allSettled when partial success is acceptable. Always add context to error messages.

## Resources

- [MDN: Handling Rejected Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/catch) — Promise.catch() reference

---

> 📘 *This lesson is part of the [Async JavaScript: Promises & Patterns](https://stanza.dev/courses/javascript-async-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*