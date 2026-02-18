---
source_course: "react-hooks-deep-dive"
source_lesson: "react-hooks-deep-dive-usedeferredvalue"
---

# useDeferredValue: Showing Stale Content

`useDeferredValue` lets you defer updating a part of the UI. It's like debouncing, but integrated with React's rendering.

## Basic Usage

```tsx
function SearchResults({ query }: { query: string }) {
  // deferredQuery lags behind query during typing
  const deferredQuery = useDeferredValue(query);
  const isStale = query !== deferredQuery;

  return (
    <div style={{ opacity: isStale ? 0.5 : 1 }}>
      <Results query={deferredQuery} />
    </div>
  );
}
```

## How It Differs from useTransition

| Aspect | useTransition | useDeferredValue |
|--------|---------------|------------------|
| Controls | State update | Value |
| Use when | You control the state | Value comes from props |
| Returns | [isPending, startTransition] | Deferred value |

## When to Use useDeferredValue

1. **Props from parent** - when you can't wrap the state update
2. **Third-party components** - when you don't control the state
3. **URL-driven state** - query params, etc.

## Example: Search with External State

```tsx
// Parent controls the query state
function SearchPage() {
  const [query, setQuery] = useState('');

  return (
    <>
      <SearchInput value={query} onChange={setQuery} />
      <SearchResults query={query} />
    </>
  );
}

// Child defers the expensive render
function SearchResults({ query }: { query: string }) {
  const deferredQuery = useDeferredValue(query);
  
  // Memoize to actually skip re-rendering with stale value
  const results = useMemo(
    () => <SlowList query={deferredQuery} />,
    [deferredQuery]
  );

  return (
    <div style={{ opacity: query !== deferredQuery ? 0.7 : 1 }}>
      {results}
    </div>
  );
}
```

## Combining with Suspense

```tsx
function App() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);

  return (
    <>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
      />
      <Suspense fallback={<Loading />}>
        <SearchResults query={deferredQuery} />
      </Suspense>
    </>
  );
}
```

When `query` changes:
1. Input updates immediately (responsive typing)
2. `deferredQuery` keeps the old value briefly
3. React renders with the new `deferredQuery` in the background
4. Once ready, React commits the new UI

## Resources

- [useDeferredValue API Reference](https://react.dev/reference/react/useDeferredValue) — Official React documentation for useDeferredValue hook

---

> 📘 *This lesson is part of the [React Hooks Deep Dive](https://stanza.dev/courses/react-hooks-deep-dive) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*