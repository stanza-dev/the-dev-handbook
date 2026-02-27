---
source_course: "javascript-async-patterns"
source_lesson: "javascript-async-patterns-async-await"
---

# Async/Await

## Introduction

Async/await is syntactic sugar over Promises that makes async code read like synchronous code. It's easier to reason about, debug, and maintain. Understanding it is essential for modern JavaScript development.

## Key Concepts

**async function**: A function that implicitly returns a Promise.

**await**: Pauses execution until a Promise settles, then returns its value.

**Top-level await**: await outside async functions (in modules only).

## Real World Context

Modern codebases use async/await almost exclusively. React data fetching, Express handlers, CLI tools—async/await is the standard.

## Deep Dive

### Basic Syntax

```javascript
async function fetchUser(id) {
  const response = await fetch(`/api/users/${id}`);
  const user = await response.json();
  return user;  // Implicitly wrapped in Promise.resolve()
}

// Usage
fetchUser(1).then(user => console.log(user));
// Or
const user = await fetchUser(1);  // In async context
```

### Async Functions Always Return Promises

```javascript
async function returnValue() {
  return 42;  // Same as Promise.resolve(42)
}

async function throwError() {
  throw new Error('Oops');  // Same as Promise.reject()
}

returnValue().then(console.log);  // 42
throwError().catch(console.error);  // Error: Oops
```

### Error Handling with Try/Catch

```javascript
async function fetchData() {
  try {
    const response = await fetch('/api/data');
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Fetch failed:', error);
    throw error;  // Re-throw to let caller handle
  } finally {
    hideLoadingSpinner();
  }
}
```

### Sequential vs Parallel

```javascript
// Sequential - slow!
async function sequential() {
  const user = await fetchUser();     // Wait...
  const posts = await fetchPosts();   // Then wait...
  return { user, posts };             // Total: user + posts time
}

// Parallel - fast!
async function parallel() {
  const [user, posts] = await Promise.all([
    fetchUser(),
    fetchPosts()
  ]);
  return { user, posts };  // Total: max(user, posts) time
}
```

### Arrow Functions and Methods

```javascript
// Arrow functions
const fetchUser = async (id) => {
  return await fetch(`/api/users/${id}`);
};

// Class methods
class UserService {
  async getUser(id) {
    return await this.api.get(`/users/${id}`);
  }
}

// Object methods
const api = {
  async fetch(url) {
    return await fetch(url);
  }
};
```

### Top-Level Await (ES2022, Modules Only)

```javascript
// In ES modules (.mjs or type="module")
const config = await fetch('/config.json').then(r => r.json());
export { config };

// The importing module waits for this to resolve
```

## Common Pitfalls

1. **Sequential when parallel is possible**: Use Promise.all for independent operations.
2. **await in loops**: Often sequential; consider Promise.all with map.
3. **Forgetting try/catch**: Errors go unhandled.

## Best Practices

- **Use try/catch for error handling**: Not .catch() on the await.
- **Parallelize independent operations**: Promise.all or Promise.allSettled.
- **Don't await in sequence unnecessarily**: Identify independent operations.
- **Always handle errors**: At least at the top level.

## Summary

Async functions implicitly return Promises. await pauses until a Promise settles. Use try/catch for error handling. Parallelize independent operations with Promise.all. Top-level await works in ES modules.

## Resources

- [MDN: async function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) — async function reference
- [MDN: await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await) — await operator reference

---

> 📘 *This lesson is part of the [Async JavaScript: Promises & Patterns](https://stanza.dev/courses/javascript-async-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*