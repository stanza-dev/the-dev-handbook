---
source_course: "react"
source_lesson: "react-useeffect-dependencies"
---

# useEffect Dependencies

## Introduction

The dependency array is the control mechanism that determines when your effect runs. Getting it right is one of the most important skills in React development. A correct dependency array prevents infinite loops, stale data, and unnecessary work. This lesson explains every variation and how to reason about dependencies.

## Key Concepts

- **Empty Array `[]`**: The effect runs once after the initial render (mount) and the cleanup runs on unmount.
- **Specific Dependencies `[a, b]`**: The effect runs after mount and again whenever any listed dependency changes.
- **No Array**: The effect runs after every single render (rarely desired).
- **Reactive Values**: Props, state, and any variables derived from them that might change between renders.

## Real World Context

A stock ticker component receives a `symbol` prop and subscribes to price updates. The dependency array should contain `[symbol]` so that when the user switches from AAPL to GOOGL, the old subscription is cleaned up and a new one is created. If you accidentally use `[]`, the component stays subscribed to the original symbol forever.

## Deep Dive

### Dependency Array Variations

```tsx
// Runs once on mount, cleanup on unmount
useEffect(() => {
  window.addEventListener('online', handleOnline);
  return () => window.removeEventListener('online', handleOnline);
}, []);

// Runs when userId changes
useEffect(() => {
  fetchUserData(userId);
}, [userId]);

// Runs after every render (usually a mistake)
useEffect(() => {
  console.log('Rendered!');
});
```

### How React Compares Dependencies

React uses `Object.is()` to compare each dependency with its previous value. If any dependency has changed, the effect re-runs:

```tsx
function SearchResults({ query, page }: { query: string; page: number }) {
  const [results, setResults] = useState([]);

  useEffect(() => {
    const controller = new AbortController();

    fetch(`/api/search?q=${query}&page=${page}`, {
      signal: controller.signal,
    })
      .then(res => res.json())
      .then(data => setResults(data.items))
      .catch(err => {
        if (err.name !== 'AbortError') console.error(err);
      });

    return () => controller.abort();
  }, [query, page]); // Re-fetches when query OR page changes

  return <ul>{results.map(r => <li key={r.id}>{r.title}</li>)}</ul>;
}
```

### Object and Function Dependencies

Objects and functions create new references on each render, which means they would cause the effect to re-run every time:

```tsx
// Problem: options is a new object every render
function Chart({ data }) {
  const options = { animate: true, color: 'blue' };

  useEffect(() => {
    drawChart(data, options);
  }, [data, options]); // options changes every render!
}

// Solution: move the object inside the effect or memoize it
function Chart({ data }) {
  useEffect(() => {
    const options = { animate: true, color: 'blue' };
    drawChart(data, options);
  }, [data]); // options is now stable
}
```

### The Exhaustive Deps Rule

The `react-hooks/exhaustive-deps` ESLint rule checks that every reactive value used inside the effect is listed in the dependency array. Always trust this rule — suppressing it leads to subtle bugs.

## Common Pitfalls

1. **Infinite loops from objects in dependencies** — Creating an object or array during render and including it in dependencies causes the effect to run every render because the reference changes. Move the creation inside the effect or memoize with `useMemo`.
2. **Suppressing the ESLint rule** — Adding `// eslint-disable-next-line` to hide dependency warnings almost always introduces bugs. Instead, restructure the code so the dependencies are correct.

## Best Practices

1. **List all reactive values** — Every prop, state variable, or derived value used inside the effect must appear in the dependency array.
2. **Move static values inside the effect** — If a value does not change between renders (a constant object, a function that does not depend on props/state), move it inside the effect body so it does not need to be a dependency.

## Summary

- The dependency array controls when the effect re-runs: empty for mount-only, specific values for change-driven, omitted for every render.
- React compares dependencies with `Object.is()`, so object and function references must be stabilized to avoid unnecessary re-runs.
- Always follow the exhaustive-deps ESLint rule to prevent stale data and infinite loop bugs.

## Code Examples

**WebSocket subscription that reconnects when the symbol dependency changes**

```tsx
function StockTicker({ symbol }: { symbol: string }) {
  const [price, setPrice] = useState<number | null>(null);

  useEffect(() => {
    const ws = new WebSocket(`wss://api.stocks.com/${symbol}`);

    ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      setPrice(data.price);
    };

    return () => ws.close();
  }, [symbol]); // Reconnects when symbol changes

  return <div>{symbol}: {price ? `$${price}` : 'Loading...'}</div>;
}
```


## Resources

- [Lifecycle of Reactive Effects](https://react.dev/learn/lifecycle-of-reactive-effects) — Official deep dive on how effect dependencies work
- [Removing Effect Dependencies](https://react.dev/learn/removing-effect-dependencies) — Official guide on reducing unnecessary dependencies

---

> 📘 *This lesson is part of the [React Basics](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*