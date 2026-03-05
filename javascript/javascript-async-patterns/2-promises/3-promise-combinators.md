---
source_course: "javascript-async-patterns"
source_lesson: "javascript-async-patterns-promise-combinators"
---

# Promise Combinators

## Introduction

Promise.all, Promise.race, Promise.allSettled, and Promise.any let you orchestrate multiple async operations. Each has different behavior for success and failure scenarios. Choosing the right one is key to efficient, resilient async code.

## Key Concepts

**Promise.all**: All must succeed, fails fast on first rejection.

**Promise.race**: Returns first to settle (success or failure).

**Promise.allSettled**: Waits for all, never rejects, reports all outcomes.

**Promise.any**: Returns first success, rejects only if all fail.

## Deep Dive

### Promise.all()

```javascript
// All succeed - get array of results
const [user, posts, comments] = await Promise.all([
  fetchUser(),
  fetchPosts(),
  fetchComments()
]);

// One fails - entire Promise.all rejects
Promise.all([
  Promise.resolve(1),
  Promise.reject('error'),  // This fails
  Promise.resolve(3)        // Never awaited!
]).catch(e => console.log(e));  // 'error'

// Empty array resolves immediately
Promise.all([]);  // Promise.resolve([])
```

### Promise.race()

```javascript
// First to settle wins (success OR failure)
const first = await Promise.race([
  fetch('/api/server1'),
  fetch('/api/server2')
]);

// Timeout pattern
function fetchWithTimeout(url, ms) {
  return Promise.race([
    fetch(url),
    new Promise((_, reject) => 
      setTimeout(() => reject(new Error('Timeout')), ms)
    )
  ]);
}

await fetchWithTimeout('/api/data', 5000);
```

### Promise.allSettled() (ES2020)

```javascript
// Never rejects - reports all outcomes
const results = await Promise.allSettled([
  fetch('/api/users'),
  fetch('/api/invalid'),  // Will fail
  fetch('/api/posts')
]);

// Results array:
// [
//   { status: 'fulfilled', value: Response },
//   { status: 'rejected', reason: Error },
//   { status: 'fulfilled', value: Response }
// ]

// Process results
results.forEach((result, i) => {
  if (result.status === 'fulfilled') {
    console.log(`Request ${i} succeeded:`, result.value);
  } else {
    console.log(`Request ${i} failed:`, result.reason);
  }
});
```

### Promise.any() (ES2021)

```javascript
// First SUCCESS wins (ignores rejections until all fail)
const fastest = await Promise.any([
  fetch('https://cdn1.example.com/data'),
  fetch('https://cdn2.example.com/data'),
  fetch('https://cdn3.example.com/data')
]);

// Only rejects if ALL fail
Promise.any([
  Promise.reject('error1'),
  Promise.reject('error2')
]).catch(e => {
  // AggregateError with all errors
  console.log(e.errors);  // ['error1', 'error2']
});
```

### Comparison Table

```
| Method        | Resolves when...        | Rejects when...        |
|---------------|-------------------------|------------------------|
| all()         | All fulfill             | First rejects          |
| race()        | First settles           | First rejects          |
| allSettled()  | All settle              | Never                  |
| any()         | First fulfills          | All reject             |
```

### Practical Patterns

```javascript
// Load essential + optional data
async function loadPage() {
  // Essential data - all must succeed
  const [user, permissions] = await Promise.all([
    fetchUser(),
    fetchPermissions()
  ]);
  
  // Optional data - don't fail if these fail
  const optional = await Promise.allSettled([
    fetchRecommendations(),
    fetchNotifications()
  ]);
  
  return { user, permissions, optional };
}
```

## Common Pitfalls

1. **Using Promise.all for independent operations that shouldn't fail together**: Use allSettled.
2. **Forgetting AggregateError with any()**: It's a special error type.
3. **Race conditions with Promise.race**: The "loser" still runs.

## Best Practices

- **Use all() for dependent operations**: Fail fast.
- **Use allSettled() for independent operations**: Don't let one failure break all.
- **Use race() for timeouts**: Not for "use fastest result".
- **Use any() for redundant sources**: CDN fallbacks.

## Summary

Promise.all() fails fast on first rejection. Promise.race() returns first to settle. Promise.allSettled() never rejects, reports all outcomes. Promise.any() returns first success. Choose based on whether operations are dependent or independent.

## Code Examples

**Promise.all()**

```javascript
// All succeed - get array of results
const [user, posts, comments] = await Promise.all([
  fetchUser(),
  fetchPosts(),
  fetchComments()
]);

// One fails - entire Promise.all rejects
Promise.all([
  Promise.resolve(1),
  Promise.reject('error'),  // This fails
  Promise.resolve(3)        // Never awaited!
]).catch(e => console.log(e));  // 'error'

// Empty array resolves immediately
Promise.all([]);  // Promise.resolve([])
```

**Promise.race()**

```javascript
// First to settle wins (success OR failure)
const first = await Promise.race([
  fetch('/api/server1'),
  fetch('/api/server2')
]);

// Timeout pattern
function fetchWithTimeout(url, ms) {
  return Promise.race([
    fetch(url),
    new Promise((_, reject) => 
      setTimeout(() => reject(new Error('Timeout')), ms)
    )
  ]);
}

await fetchWithTimeout('/api/data', 5000);
```


## Resources

- [MDN: Promise.all()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all) — Promise.all reference
- [MDN: Promise.allSettled()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/allSettled) — Promise.allSettled reference

---

> 📘 *This lesson is part of the [Async JavaScript: Promises & Patterns](https://stanza.dev/courses/javascript-async-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*