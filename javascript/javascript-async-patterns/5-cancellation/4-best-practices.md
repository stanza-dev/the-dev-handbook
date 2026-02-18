---
source_course: "javascript-async-patterns"
source_lesson: "javascript-async-patterns-cancellation-best-practices"
---

# Cancellation Best Practices

## Introduction

Proper cancellation handling separates robust applications from fragile ones. This lesson covers patterns that prevent memory leaks, race conditions, and poor UX.

## Deep Dive

### Always Handle AbortError

```javascript
// Bad: treats cancellation as error
async function badFetch(url, signal) {
  try {
    return await fetch(url, { signal });
  } catch (error) {
    showErrorToUser(error);  // Shows error for cancelled requests!
  }
}

// Good: separate handling
async function goodFetch(url, signal) {
  try {
    return await fetch(url, { signal });
  } catch (error) {
    if (error.name === 'AbortError') {
      // Cancellation is expected, not an error
      return null;
    }
    throw error;  // Real errors propagate
  }
}
```

### Prevent State Updates After Cancel

```javascript
// React pattern
function DataLoader({ id }) {
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    let cancelled = false;  // Or use AbortController
    
    fetchData(id)
      .then(result => {
        if (!cancelled) setData(result);
      })
      .catch(err => {
        if (!cancelled) setError(err);
      });
    
    return () => { cancelled = true; };
  }, [id]);
}
```

### Race Condition Prevention

```javascript
// Problem: later request returns before earlier one
let latestRequestId = 0;

async function fetchWithRaceProtection(url) {
  const requestId = ++latestRequestId;
  const response = await fetch(url);
  const data = await response.json();
  
  if (requestId !== latestRequestId) {
    return null;  // Stale response, ignore
  }
  return data;
}

// Better: use AbortController
let controller = null;

async function fetchLatest(url) {
  controller?.abort();
  controller = new AbortController();
  
  try {
    const response = await fetch(url, { signal: controller.signal });
    return await response.json();
  } catch (e) {
    if (e.name === 'AbortError') return null;
    throw e;
  }
}
```

### Testing Cancellation

```javascript
test('cancels request on abort', async () => {
  const controller = new AbortController();
  const fetchPromise = myFetch('/api/slow', controller.signal);
  
  controller.abort();
  
  await expect(fetchPromise).rejects.toThrow('AbortError');
});

test('ignores cancelled requests', async () => {
  const controller = new AbortController();
  const setData = jest.fn();
  
  loadData(setData, controller.signal);
  controller.abort();
  
  await delay(100);
  expect(setData).not.toHaveBeenCalled();
});
```

## Summary

Always distinguish AbortError from real errors. Prevent state updates after cancellation. Use abort instead of stale flags for race conditions. Test cancellation behavior. Clean up in every code path.

---

> 📘 *This lesson is part of the [Async JavaScript: Promises & Patterns](https://stanza.dev/courses/javascript-async-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*