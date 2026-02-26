---
source_course: "javascript-async-patterns"
source_lesson: "javascript-async-patterns-abort-controller"
---

# AbortController Fundamentals

## Introduction

Async operations can outlive their usefulness—a user navigates away, a timeout expires, or new data supersedes old requests. AbortController provides a standard way to cancel fetch requests and other async operations. It's essential for responsive, resource-efficient applications.

## Key Concepts

**AbortController**: Object that can signal cancellation to async operations.

**AbortSignal**: The signal object passed to cancellable operations.

**AbortError**: Error thrown when an operation is aborted.

## Real World Context

Type-ahead search (cancel pending requests), route changes in SPAs, request timeouts, cleanup in React useEffect—cancellation is everywhere in modern apps.

## Deep Dive

### Basic Usage

```javascript
const controller = new AbortController();
const { signal } = controller;

fetch('/api/data', { signal })
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => {
    if (error.name === 'AbortError') {
      console.log('Request was cancelled');
    } else {
      console.error('Fetch failed:', error);
    }
  });

// Later: cancel the request
controller.abort();
```

### Abort with Reason

```javascript
const controller = new AbortController();

controller.abort('User cancelled');  // Custom reason
controller.abort(new Error('Timeout'));  // Error object

// Access the reason
controller.signal.reason;  // 'User cancelled'
```

### Checking Signal State

```javascript
const controller = new AbortController();
const { signal } = controller;

signal.aborted;  // false
controller.abort();
signal.aborted;  // true

// Listen for abort
signal.addEventListener('abort', () => {
  console.log('Signal aborted:', signal.reason);
});
```

### With Async/Await

```javascript
async function fetchWithCancellation(url, signal) {
  try {
    const response = await fetch(url, { signal });
    return await response.json();
  } catch (error) {
    if (error.name === 'AbortError') {
      console.log('Fetch cancelled');
      return null;
    }
    throw error;
  }
}

const controller = new AbortController();
const data = await fetchWithCancellation('/api/data', controller.signal);
```

## Common Pitfalls

1. **Creating new controller for each request in a loop**: Reuse or create appropriate scope.
2. **Not checking for AbortError**: Treating it as a real failure.
3. **Forgetting to abort in cleanup**: Leads to memory leaks.

## Best Practices

- **Always check error.name**: Distinguish AbortError from other failures.
- **Abort in cleanup functions**: useEffect return, component unmount.
- **Consider abort reason**: Helps debugging.
- **Reuse signals for related requests**: One controller can cancel many.

## Summary

AbortController creates a signal that can cancel fetch and other operations. Check `error.name === 'AbortError'` to distinguish cancellation. Always clean up by aborting pending requests when components unmount or context changes.

## Code Examples

**Basic Usage**

```javascript
const controller = new AbortController();
const { signal } = controller;

fetch('/api/data', { signal })
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => {
    if (error.name === 'AbortError') {
      console.log('Request was cancelled');
    } else {
      console.error('Fetch failed:', error);
    }
  });

// Later: cancel the request
controller.abort();
```

**Abort with Reason**

```javascript
const controller = new AbortController();

controller.abort('User cancelled');  // Custom reason
controller.abort(new Error('Timeout'));  // Error object

// Access the reason
controller.signal.reason;  // 'User cancelled'
```


## Resources

- [MDN: AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController) — AbortController reference
- [MDN: AbortSignal](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal) — AbortSignal reference

---

> 📘 *This lesson is part of the [Async JavaScript: Promises & Patterns](https://stanza.dev/courses/javascript-async-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*