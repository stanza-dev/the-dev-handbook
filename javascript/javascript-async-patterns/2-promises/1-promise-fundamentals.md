---
source_course: "javascript-async-patterns"
source_lesson: "javascript-async-patterns-promise-fundamentals"
---

# Promise Fundamentals

## Introduction

Promises revolutionized JavaScript async programming. A Promise represents a value that may not exist yet—the eventual result of an async operation. Unlike callbacks, promises can be chained, composed, and handled with cleaner error management.

## Key Concepts

**Promise**: An object representing the eventual completion or failure of an async operation.

**States**: pending (initial), fulfilled (success), rejected (failure). Once settled, cannot change.

**Thenable**: Any object with a `.then()` method.

## Real World Context

Every fetch() returns a Promise. APIs return promises. Modern libraries use promises. Understanding them is required for any async work in JavaScript.

## Deep Dive

### Promise States

```javascript
// Pending -> Fulfilled
const fulfilled = new Promise((resolve, reject) => {
  setTimeout(() => resolve('Success!'), 1000);
});

// Pending -> Rejected
const rejected = new Promise((resolve, reject) => {
  setTimeout(() => reject(new Error('Failed!')), 1000);
});

// State is permanent once settled
const promise = new Promise((resolve) => {
  resolve('first');   // This wins
  resolve('second');  // Ignored
  reject('error');    // Ignored
});
```

### Creating Promises

```javascript
// Constructor
const promise = new Promise((resolve, reject) => {
  const success = doAsyncWork();
  if (success) {
    resolve(result);
  } else {
    reject(new Error('Failed'));
  }
});

// Static methods for immediate values
Promise.resolve('immediate value');
Promise.reject(new Error('immediate error'));
```

### Consuming with .then() and .catch()

```javascript
fetch('/api/user')
  .then(response => response.json())  // Returns new promise
  .then(user => {
    console.log('User:', user);
    return user.id;  // Passed to next .then()
  })
  .then(userId => fetch(`/api/posts/${userId}`))
  .catch(error => {
    // Catches ANY error in the chain
    console.error('Failed:', error);
  })
  .finally(() => {
    // Always runs, no value passed
    hideLoadingSpinner();
  });
```

### Promise Chaining

```javascript
// Each .then() returns a new Promise
Promise.resolve(1)
  .then(x => x + 1)    // 2
  .then(x => x * 2)    // 4
  .then(x => x + 10)   // 14
  .then(console.log);  // Logs 14

// Returning a Promise waits for it
Promise.resolve('start')
  .then(() => fetch('/api/data'))  // Returns Promise
  .then(response => response.json())  // Waits for fetch
  .then(data => console.log(data));
```

### Error Propagation

```javascript
Promise.resolve('start')
  .then(() => { throw new Error('Oops!'); })
  .then(() => console.log('Skipped'))  // Skipped!
  .then(() => console.log('Also skipped'))  // Skipped!
  .catch(e => console.log('Caught:', e.message))  // 'Caught: Oops!'
  .then(() => console.log('Continues'));  // Runs!
```

## Common Pitfalls

1. **Forgetting to return in .then()**: Next handler gets undefined.
2. **Nested promises**: Use return, not nested .then().
3. **Missing .catch()**: Unhandled rejection warnings.

## Best Practices

- **Always return in .then()**: Chain properly.
- **End chains with .catch()**: Handle all errors.
- **Use .finally() for cleanup**: Loading spinners, etc.
- **Prefer async/await for readability**: When possible.

## Summary

Promises represent async operation results with three states: pending, fulfilled, rejected. Chain with .then(), handle errors with .catch(), clean up with .finally(). Always return values in .then() handlers. Errors propagate down the chain until caught.

## Code Examples

**Promise States**

```javascript
// Pending -> Fulfilled
const fulfilled = new Promise((resolve, reject) => {
  setTimeout(() => resolve('Success!'), 1000);
});

// Pending -> Rejected
const rejected = new Promise((resolve, reject) => {
  setTimeout(() => reject(new Error('Failed!')), 1000);
});

// State is permanent once settled
const promise = new Promise((resolve) => {
  resolve('first');   // This wins
  resolve('second');  // Ignored
  reject('error');    // Ignored
});
```

**Creating Promises**

```javascript
// Constructor
const promise = new Promise((resolve, reject) => {
  const success = doAsyncWork();
  if (success) {
    resolve(result);
  } else {
    reject(new Error('Failed'));
  }
});

// Static methods for immediate values
Promise.resolve('immediate value');
Promise.reject(new Error('immediate error'));
```


## Resources

- [MDN: Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) — Promise reference
- [MDN: Using Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises) — Guide to using promises

---

> 📘 *This lesson is part of the [Async JavaScript: Promises & Patterns](https://stanza.dev/courses/javascript-async-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*