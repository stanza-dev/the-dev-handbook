---
source_course: "react"
source_lesson: "react-useeffect-cleanup"
---

# Cleanup Functions

## Introduction

Many side effects create resources that need to be released: event listeners, timers, network connections, subscriptions. If you do not clean up, your application will leak memory, make duplicate requests, or respond to events from components that are no longer on screen. The cleanup function is React's mechanism for tearing down what the effect set up.

## Key Concepts

- **Cleanup Function**: A function returned from the effect callback that React calls to undo the effect's work.
- **Cleanup Timing**: React runs cleanup before re-running the effect (when dependencies change) and when the component unmounts.
- **AbortController**: The standard Web API for cancelling in-flight fetch requests during cleanup.
- **Strict Mode Double-Invocation**: In development, React intentionally runs effects twice (setup, cleanup, setup) to help you find missing cleanup logic.

## Real World Context

A music streaming app has a `NowPlaying` component that updates the MediaSession API (browser media controls). When the user navigates away from the player, the cleanup function resets the MediaSession so stale metadata does not appear in the OS media controls. Without cleanup, switching songs rapidly could show the wrong album art.

## Deep Dive

### Event Listener Cleanup

```tsx
function ScrollProgress() {
  const [progress, setProgress] = useState(0);

  useEffect(() => {
    const handleScroll = () => {
      const scrollTop = document.documentElement.scrollTop;
      const scrollHeight = document.documentElement.scrollHeight - window.innerHeight;
      setProgress(scrollHeight > 0 ? (scrollTop / scrollHeight) * 100 : 0);
    };

    window.addEventListener('scroll', handleScroll);

    return () => {
      window.removeEventListener('scroll', handleScroll);
    };
  }, []);

  return <div className="progress-bar" style={{ width: `${progress}%` }} />;
}
```

### Timer Cleanup

```tsx
function Countdown({ seconds }: { seconds: number }) {
  const [remaining, setRemaining] = useState(seconds);

  useEffect(() => {
    if (remaining <= 0) return;

    const timerId = setInterval(() => {
      setRemaining(prev => prev - 1);
    }, 1000);

    return () => clearInterval(timerId);
  }, [remaining]);

  return <p>{remaining > 0 ? `${remaining}s left` : 'Done!'}</p>;
}
```

### Fetch Request Cleanup with AbortController

```tsx
function SearchResults({ query }: { query: string }) {
  const [results, setResults] = useState([]);

  useEffect(() => {
    if (!query) return;

    const controller = new AbortController();

    fetch(`/api/search?q=${encodeURIComponent(query)}`, {
      signal: controller.signal,
    })
      .then(res => res.json())
      .then(data => setResults(data))
      .catch(err => {
        if (err.name !== 'AbortError') {
          console.error('Search failed:', err);
        }
      });

    return () => controller.abort();
  }, [query]);

  return <ul>{results.map(r => <li key={r.id}>{r.title}</li>)}</ul>;
}
```

When `query` changes quickly (user typing fast), each keystroke aborts the previous in-flight request so only the latest result is applied.

### Why Strict Mode Runs Effects Twice

In development, React 18+ intentionally calls your effect, then the cleanup, then the effect again. This simulates what happens if a component is remounted and helps you catch missing cleanup:

```
// Development behavior (Strict Mode):
// 1. Effect runs (setup)
// 2. Cleanup runs (teardown)
// 3. Effect runs again (setup)

// Production behavior:
// 1. Effect runs (setup)
```

If your effect works correctly with this pattern, it is correctly handling cleanup.

## Common Pitfalls

1. **Forgetting to clean up intervals** — A `setInterval` without cleanup continues running after the component unmounts, causing memory leaks and errors when it tries to update state on an unmounted component.
2. **Not aborting fetch requests** — Without `AbortController`, a slow request can resolve after the component has unmounted or after the dependency has changed, potentially applying stale data.

## Best Practices

1. **Always clean up subscriptions and timers** — If your effect calls `addEventListener`, `setInterval`, `setTimeout`, or opens a connection, return a cleanup function.
2. **Use AbortController for fetch requests** — This is the standard pattern for cancellable network requests and works with the native `fetch` API.

## Summary

- Return a cleanup function from `useEffect` to release resources created by the effect.
- Cleanup runs before each effect re-run (when dependencies change) and when the component unmounts.
- Use `AbortController` for fetch cleanup and always clear timers and event listeners.

## Code Examples

**Window resize listener with proper cleanup**

```tsx
function WindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    const handleResize = () => {
      setSize({ width: window.innerWidth, height: window.innerHeight });
    };

    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return <p>{size.width} x {size.height}</p>;
}
```


## Resources

- [Synchronizing with Effects](https://react.dev/learn/synchronizing-with-effects) — Official guide on effect setup and cleanup

---

> 📘 *This lesson is part of the [React Basics](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*