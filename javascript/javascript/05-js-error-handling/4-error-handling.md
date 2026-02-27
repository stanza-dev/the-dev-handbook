---
source_course: "javascript"
source_lesson: "javascript-async-error-handling"
---

# Async Error Handling

## Introduction

Async code introduces new error handling challenges. Errors in callbacks, promises, and async/await each require different approaches. Understanding these patterns is essential for building reliable async applications.

## Key Concepts

**Rejection**: The promise equivalent of throwing an error.

**.catch()**: Promise method for handling rejections.

**Unhandled Rejection**: A rejected promise without a catch handler.

## Real World Context

API calls, database queries, file operations—most async operations can fail. Proper async error handling prevents crashes and provides meaningful feedback.

## Deep Dive

### Promise Error Handling

```javascript
// .catch() method
fetch('/api/data')
  .then(response => response.json())
  .then(data => process(data))
  .catch(error => {
    console.error('Request failed:', error);
  });

// catch handles errors from any previous .then()
fetch('/api/data')
  .then(response => {
    if (!response.ok) throw new Error('HTTP error');
    return response.json();
  })
  .then(data => process(data))
  .catch(error => {
    // Catches both network errors and our thrown error
    console.error(error);
  });
```

### Async/Await Error Handling

```javascript
// try/catch with async/await
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
  }
}
```

### Handling Multiple Async Operations

```javascript
// Promise.all - fails fast on first error
try {
  const [users, posts] = await Promise.all([
    fetchUsers(),
    fetchPosts()
  ]);
} catch (error) {
  // One or both failed
}

// Promise.allSettled - get all results
const results = await Promise.allSettled([
  fetchUsers(),
  fetchPosts(),
  fetchComments()
]);

results.forEach((result, i) => {
  if (result.status === 'fulfilled') {
    console.log('Success:', result.value);
  } else {
    console.log('Failed:', result.reason);
  }
});
```

### Common Patterns

```javascript
// Wrapper for async route handlers
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

app.get('/users', asyncHandler(async (req, res) => {
  const users = await db.getUsers();  // Errors auto-forwarded
  res.json(users);
}));

// Retry pattern
async function fetchWithRetry(url, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      return await fetch(url).then(r => r.json());
    } catch (error) {
      if (i === retries - 1) throw error;
      await new Promise(r => setTimeout(r, 1000 * (i + 1)));
    }
  }
}
```

### Don't Forget to Await!

```javascript
// Bug: unhandled rejection
async function processData() {
  fetchData();  // Forgot await! Error is lost
}

// Correct
async function processData() {
  await fetchData();  // Errors will propagate
}

// Also correct: explicit handling
async function processData() {
  fetchData().catch(console.error);  // Fire and forget with handling
}
```

## Common Pitfalls

1. **Missing await**: Unhandled promise, error is lost.
2. **catch returning undefined**: Downstream code gets undefined instead of error.
3. **Not re-throwing in catch**: Masks errors from callers.

## Best Practices

- **Always await or handle promises**: No floating promises.
- **Use try/catch with async/await**: Clean error handling.
- **Use Promise.allSettled for independent operations**: Don't let one failure break all.
- **Re-throw errors you can't handle**: Let them propagate.

## Summary

Use `.catch()` for promise chains and `try/catch` for async/await. Always await or handle promises to avoid unhandled rejections. Use `Promise.allSettled` when you need all results regardless of individual failures. Never forget the `await` keyword.

## Resources

- [MDN: Using Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises) — Promise error handling guide
- [MDN: Promise.prototype.catch()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/catch) — catch() method reference

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*