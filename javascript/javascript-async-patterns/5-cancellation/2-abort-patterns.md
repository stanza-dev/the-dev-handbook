---
source_course: "javascript-async-patterns"
source_lesson: "javascript-async-patterns-abort-patterns"
---

# Cancellation Patterns

## Introduction

Beyond basic fetch cancellation, AbortController enables powerful patterns: timeouts, race conditions, batch cancellation, and custom cancellable operations. These patterns are essential for production applications.

## Deep Dive

### Timeout Pattern

```javascript
// Using AbortSignal.timeout() (modern browsers)
fetch('/api/data', { 
  signal: AbortSignal.timeout(5000)  // 5 second timeout
});

// Manual timeout
function fetchWithTimeout(url, timeout = 5000) {
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort('Timeout'), timeout);
  
  return fetch(url, { signal: controller.signal })
    .finally(() => clearTimeout(timeoutId));
}
```

### Combining Signals

```javascript
// AbortSignal.any() - abort if ANY signal aborts
const controller = new AbortController();
const timeoutSignal = AbortSignal.timeout(5000);

const combinedSignal = AbortSignal.any([
  controller.signal,
  timeoutSignal
]);

fetch('/api/data', { signal: combinedSignal });

// Can be cancelled by timeout OR manual abort
controller.abort();  // Cancels immediately
```

### Search/Autocomplete Pattern

```javascript
class SearchController {
  #controller = null;
  
  async search(query) {
    // Cancel previous request
    this.#controller?.abort();
    this.#controller = new AbortController();
    
    try {
      const response = await fetch(
        `/api/search?q=${encodeURIComponent(query)}`,
        { signal: this.#controller.signal }
      );
      return await response.json();
    } catch (error) {
      if (error.name === 'AbortError') {
        return null;  // Cancelled, ignore
      }
      throw error;
    }
  }
}

const search = new SearchController();
input.addEventListener('input', async (e) => {
  const results = await search.search(e.target.value);
  if (results) displayResults(results);
});
```

### React useEffect Cleanup

```javascript
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    const controller = new AbortController();
    
    fetch(`/api/users/${userId}`, { signal: controller.signal })
      .then(r => r.json())
      .then(setUser)
      .catch(err => {
        if (err.name !== 'AbortError') {
          console.error(err);
        }
      });
    
    return () => controller.abort();  // Cleanup
  }, [userId]);
  
  return user ? <div>{user.name}</div> : <Loading />;
}
```

### Custom Cancellable Operation

```javascript
function cancellableDelay(ms, signal) {
  return new Promise((resolve, reject) => {
    const timeoutId = setTimeout(resolve, ms);
    
    signal?.addEventListener('abort', () => {
      clearTimeout(timeoutId);
      reject(new DOMException('Delay aborted', 'AbortError'));
    });
  });
}

const controller = new AbortController();
cancellableDelay(5000, controller.signal)
  .then(() => console.log('Completed'))
  .catch(e => console.log('Cancelled'));

controller.abort();  // Rejects immediately
```

## Summary

Use `AbortSignal.timeout()` for simple timeouts. `AbortSignal.any()` combines multiple signals. Cancel previous requests for search/autocomplete. Always abort in React cleanup. Make custom operations cancellable by listening to the signal.

## Resources

- [MDN: AbortSignal.timeout()](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal/timeout_static) — Timeout helper method

---

> 📘 *This lesson is part of the [Async JavaScript: Promises & Patterns](https://stanza.dev/courses/javascript-async-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*