---
source_course: "react-intermediate"
source_lesson: "react-fetch-error-handling"
---

# Fetch Error Handling

## Introduction

Network requests can fail in many ways: the server returns an error, the network is unreliable, the request times out, or the response contains unexpected data. Robust error handling covers all these cases and provides actionable feedback to users.

## Key Concepts

- **HTTP error responses**: The Fetch API does not reject on HTTP error status codes (404, 500). You must check \`response.ok\` manually.
- **Network errors**: When the network is down or the server is unreachable, fetch rejects the promise with a TypeError.
- **Timeout handling**: The Fetch API does not have a built-in timeout, but you can implement one with AbortController and setTimeout.

## Real World Context

A mobile user on a spotty connection tries to load their order history. The first request times out after 10 seconds. The app shows a "Connection slow - tap to retry" message. The user taps retry, the second attempt succeeds, and the orders appear. Without proper error handling, the user would see an infinite spinner.

## Deep Dive

A critical gotcha with the Fetch API is that HTTP error responses (4xx, 5xx) do NOT cause the promise to reject. The promise resolves with the error response, and you must check \`response.ok\` yourself:

\`\`\`tsx
async function fetchWithErrorHandling(url: string, signal?: AbortSignal) {
  const response = await fetch(url, { signal });

  if (!response.ok) {
    const body = await response.text();
    throw new Error(
      \`HTTP \${response.status}: \${body || response.statusText}\`
    );
  }

  return response.json();
}
\`\`\`

For timeouts, combine AbortController with setTimeout:

\`\`\`tsx
function fetchWithTimeout(url: string, timeoutMs: number = 10000) {
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), timeoutMs);

  return fetch(url, { signal: controller.signal })
    .then(res => {
      clearTimeout(timeoutId);
      if (!res.ok) throw new Error(\`HTTP \${res.status}\`);
      return res.json();
    })
    .catch(err => {
      clearTimeout(timeoutId);
      if (err.name === 'AbortError') {
        throw new Error('Request timed out');
      }
      throw err;
    });
}
\`\`\`

For retry logic, implement exponential backoff to avoid overwhelming the server:

\`\`\`tsx
async function fetchWithRetry(url: string, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      return await fetchWithErrorHandling(url);
    } catch (err) {
      if (i === retries - 1) throw err;
      await new Promise(r => setTimeout(r, 1000 * Math.pow(2, i)));
    }
  }
}
\`\`\`

## Common Pitfalls

1. **Treating fetch like axios** — Fetch does not throw on HTTP errors. Always check \`response.ok\` or the status code before parsing the body.
2. **No timeout handling** — Without a timeout, a hung server causes the user to wait indefinitely. Always set a reasonable timeout.

## Best Practices

1. **Check response.ok** — Always validate the HTTP status before parsing the response body.
2. **Implement timeouts** — Use AbortController with setTimeout to enforce a maximum wait time for every request.

## Summary

- The Fetch API resolves on HTTP errors; always check \`response.ok\`.
- Use AbortController with setTimeout for timeout handling.
- Implement retry with exponential backoff for transient network failures.

## Code Examples

**Error handling with status checks and timeouts**

```tsx
async function fetchWithErrorHandling(url: string, signal?: AbortSignal) {
  const response = await fetch(url, { signal });
  if (!response.ok) {
    throw new Error(`HTTP ${response.status}: ${response.statusText}`);
  }
  return response.json();
}

function fetchWithTimeout(url: string, timeoutMs = 10000) {
  const controller = new AbortController();
  const id = setTimeout(() => controller.abort(), timeoutMs);

  return fetch(url, { signal: controller.signal })
    .then(res => { clearTimeout(id); return res; })
    .catch(err => {
      clearTimeout(id);
      throw err.name === 'AbortError'
        ? new Error('Request timed out')
        : err;
    });
}
```


## Resources

- [Using the Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) — MDN guide to the Fetch API

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*