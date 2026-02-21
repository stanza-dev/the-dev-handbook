---
source_course: "react-intermediate"
source_lesson: "react-usedeferredvalue"
---

# useDeferredValue Hook

## Introduction

While `useTransition` wraps the state update, `useDeferredValue` wraps the value itself. It's a simpler API when you want to defer a value without needing the `isPending` state.

## Key Concepts

**Signature:**
```tsx
const deferredValue = useDeferredValue(value);
```

- Takes any value and returns a "deferred" version
- During urgent updates, returns the old value first
- Then updates to the new value in a background render

## Real World Context

Perfect for:
- Search-as-you-type interfaces
- Filtering derived data from props
- Any scenario where you receive a value and need to derive expensive UI from it

## Deep Dive

### How It Works

1. You update some state (e.g., search query)
2. `useDeferredValue` initially returns the old value
3. React renders with the new state but old deferred value
4. In the background, React prepares a render with the new deferred value
5. When ready, React switches to the new render

### Comparing to useTransition

| useTransition | useDeferredValue |
|--------------|------------------|
| Wraps the setter | Wraps the value |
| Returns `isPending` | No pending indicator built-in |
| You control what to defer | Value automatically deferred |
| For state you own | For values you receive (props) |

### Detecting Stale Content

Check if the deferred value differs from the current value:

```tsx
const query = props.searchQuery;
const deferredQuery = useDeferredValue(query);
const isStale = query !== deferredQuery;

return (
  <div style={{ opacity: isStale ? 0.5 : 1 }}>
    <Results query={deferredQuery} />
  </div>
);
```

### Important: Memoization

For `useDeferredValue` to actually skip work, the child component must be memoized:

```tsx
const deferredQuery = useDeferredValue(query);

// Without memo, SlowList still re-renders!
// With memo, it only re-renders when deferredQuery changes
<SlowList query={deferredQuery} />
```

## Common Pitfalls

1. **Forgetting to memoize children**: Without `memo`, the optimization is lost because the child re-renders anyway.
2. **Using for urgent updates**: Don't use for UI that must update immediately (input values, hover states).
3. **Creating objects inline**: Passing a new object to `useDeferredValue` each render defeats the purpose.

## Best Practices

- Always wrap the component consuming the deferred value with `memo()`
- Use for props you receive, `useTransition` for state you own
- Show a visual indicator when content is stale (`isStale = current !== deferred`)
- Don't defer the input value itself - defer the derived computation
- Combine with Suspense for data fetching scenarios

## Summary

`useDeferredValue` is a simpler alternative to `useTransition` for deferring values you receive. It automatically returns old values during urgent updates, then updates in the background. Always pair with `memo()` for the optimization to work.

## Code Examples

**useDeferredValue with memoized search results**

```tsx
import { useState, useDeferredValue, memo } from 'react';

// Must be memoized for useDeferredValue to skip re-renders
const SearchResults = memo(function SearchResults({ query }: { query: string }) {
  // Expensive search/filter operation
  const results = expensiveSearch(query);
  
  return (
    <ul>
      {results.map(result => (
        <li key={result.id}>{result.title}</li>
      ))}
    </ul>
  );
});

function SearchPage() {
  const [query, setQuery] = useState('');
  
  // Deferred version of query
  const deferredQuery = useDeferredValue(query);
  
  // Detect if showing stale content
  const isStale = query !== deferredQuery;

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search..."
      />
      
      {/* Visual feedback for stale content */}
      <div style={{
        opacity: isStale ? 0.5 : 1,
        transition: 'opacity 200ms'
      }}>
        {isStale && <span className="updating">Updating...</span>}
        
        {/* Uses deferred value - won't block typing */}
        <SearchResults query={deferredQuery} />
      </div>
    </div>
  );
}
```


## Resources

- [useDeferredValue Reference](https://react.dev/reference/react/useDeferredValue) — Official React documentation for useDeferredValue

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*