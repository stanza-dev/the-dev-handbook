---
source_course: "javascript-async-patterns"
source_lesson: "javascript-async-patterns-callbacks"
---

# Callback Patterns

## Introduction

Before Promises, callbacks were the only way to handle async operations. While modern code prefers async/await, callbacks still appear everywhere—event handlers, array methods, and many Node.js APIs. Understanding them helps you work with any JavaScript codebase.

## Key Concepts

**Callback**: A function passed to another function to be called later.

**Error-First Callback**: Node.js convention where first parameter is error.

**Callback Hell**: Deeply nested callbacks that are hard to read.

## Real World Context

Event listeners, setTimeout, fs.readFile, Express middleware—callbacks are everywhere. Even if you prefer Promises, you'll encounter callback APIs regularly.

## Deep Dive

### Basic Callbacks

```javascript
function fetchData(callback) {
  setTimeout(() => {
    callback({ id: 1, name: 'Alice' });
  }, 1000);
}

fetchData((data) => {
  console.log('Got data:', data);
});
```

### Error-First Callbacks (Node.js Style)

```javascript
function readFile(path, callback) {
  setTimeout(() => {
    if (!path) {
      callback(new Error('Path required'), null);
      return;
    }
    callback(null, 'file contents');
  }, 100);
}

readFile('/data.txt', (error, data) => {
  if (error) {
    console.error('Failed:', error.message);
    return;
  }
  console.log('Data:', data);
});
```

### Callback Hell (The Problem)

```javascript
// Deeply nested, hard to follow
getUser(userId, (err, user) => {
  if (err) return handleError(err);
  getOrders(user.id, (err, orders) => {
    if (err) return handleError(err);
    getOrderDetails(orders[0].id, (err, details) => {
      if (err) return handleError(err);
      getShippingInfo(details.shipId, (err, shipping) => {
        if (err) return handleError(err);
        displayOrder(user, orders[0], details, shipping);
      });
    });
  });
});
```

### Escaping Callback Hell

```javascript
// 1. Named functions
function handleUser(err, user) {
  if (err) return handleError(err);
  getOrders(user.id, handleOrders);
}

function handleOrders(err, orders) {
  if (err) return handleError(err);
  getOrderDetails(orders[0].id, handleDetails);
}

getUser(userId, handleUser);

// 2. Promise wrapping (best solution)
function promisify(fn) {
  return (...args) => new Promise((resolve, reject) => {
    fn(...args, (err, result) => {
      if (err) reject(err);
      else resolve(result);
    });
  });
}

const getUserPromise = promisify(getUser);
```

### Common Callback Patterns

```javascript
// Array methods with callbacks
[1, 2, 3].map(n => n * 2);      // Transform
[1, 2, 3].filter(n => n > 1);   // Filter
[1, 2, 3].forEach(n => log(n)); // Side effects

// Event handlers
button.addEventListener('click', (event) => {
  console.log('Clicked!', event.target);
});

// setTimeout/setInterval
const timerId = setTimeout(() => {
  console.log('Delayed execution');
}, 1000);
clearTimeout(timerId);  // Cancel
```

## Common Pitfalls

1. **Forgetting to handle errors**: Always check error parameter.
2. **Callback not called**: Leads to hanging operations.
3. **Callback called multiple times**: Causes bugs; use once wrappers.

## Best Practices

- **Wrap in Promises**: Use `util.promisify` or custom wrapper.
- **Use named functions**: Avoid deep nesting.
- **Always handle errors**: Check error-first parameter.
- **Prefer async/await**: When working with Promises.

## Summary

Callbacks are functions passed to other functions for later execution. Node.js uses error-first convention. Callback hell is solved by named functions or promisification. Modern code prefers Promises/async-await, but callbacks remain common in events and many APIs.

## Code Examples

**Basic Callbacks**

```javascript
function fetchData(callback) {
  setTimeout(() => {
    callback({ id: 1, name: 'Alice' });
  }, 1000);
}

fetchData((data) => {
  console.log('Got data:', data);
});
```

**Error-First Callbacks (Node.js Style)**

```javascript
function readFile(path, callback) {
  setTimeout(() => {
    if (!path) {
      callback(new Error('Path required'), null);
      return;
    }
    callback(null, 'file contents');
  }, 100);
}

readFile('/data.txt', (error, data) => {
  if (error) {
    console.error('Failed:', error.message);
    return;
  }
  console.log('Data:', data);
});
```


## Resources

- [MDN: Callback function](https://developer.mozilla.org/en-US/docs/Glossary/Callback_function) — Callback function glossary
- [Node.js: util.promisify()](https://nodejs.org/api/util.html#utilpromisifyoriginal) — Converting callbacks to promises

---

> 📘 *This lesson is part of the [Async JavaScript: Promises & Patterns](https://stanza.dev/courses/javascript-async-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*