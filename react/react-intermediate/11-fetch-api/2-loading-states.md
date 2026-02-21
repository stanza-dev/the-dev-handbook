---
source_course: "react-intermediate"
source_lesson: "react-fetch-loading-states"
---

# Managing Loading States

## Introduction

A great user experience depends on clear communication about what is happening. Loading states tell the user that data is being fetched, error states explain what went wrong, and empty states clarify when there is simply no data to show. This lesson covers patterns for managing these states elegantly.

## Key Concepts

- **Loading state**: A boolean or enum indicating whether a fetch is in progress, used to show spinners, skeletons, or placeholders.
- **Error state**: Information about a failed request, used to display helpful error messages and retry options.
- **Empty state**: The case where a successful fetch returns no results, requiring a distinct UI from loading.

## Real World Context

A search results page has four distinct states: initial (no search yet), loading (search in progress), results (data returned), and empty (no matches). Without handling all four, users see confusing blank screens or stale data from a previous search.

## Deep Dive

A clean pattern is to use a discriminated union status instead of separate boolean flags. This prevents impossible states like being both loading and having an error.

\`\`\`tsx
type FetchState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: string };

function useDataFetch<T>(url: string | null) {
  const [state, setState] = useState<FetchState<T>>({ status: 'idle' });

  useEffect(() => {
    if (!url) { setState({ status: 'idle' }); return; }
    const controller = new AbortController();
    setState({ status: 'loading' });

    fetch(url, { signal: controller.signal })
      .then(res => {
        if (!res.ok) throw new Error('Request failed');
        return res.json();
      })
      .then(data => setState({ status: 'success', data }))
      .catch(err => {
        if (err.name !== 'AbortError') {
          setState({ status: 'error', error: err.message });
        }
      });

    return () => controller.abort();
  }, [url]);

  return state;
}
\`\`\`

The discriminated union approach ensures that you cannot accidentally access \`data\` while in a loading state. TypeScript narrows the type based on the status check, providing compile-time safety.

For the UI layer, skeleton screens provide a better experience than spinners because they preview the page layout, reducing layout shift when content loads.

## Common Pitfalls

1. **Using separate booleans for loading and error** — You can end up in impossible states like \`{ loading: true, error: 'something' }\`. A discriminated union prevents this.
2. **Forgetting the empty state** — An empty result array is different from loading. Show a helpful message like "No results found" instead of a blank screen.

## Best Practices

1. **Use discriminated unions** — Model fetch state as a union of mutually exclusive states for type safety.
2. **Use skeleton screens** — Replace generic spinners with layout-matching skeletons for a polished loading experience.

## Summary

- Model loading state as a discriminated union to prevent impossible states.
- Handle idle, loading, success, error, and empty states distinctly.
- Skeleton screens provide a smoother loading experience than spinners.

## Code Examples

**Discriminated union for fetch state management**

```tsx
type FetchState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: string };

function SearchResults({ query }: { query: string }) {
  const state = useDataFetch<Product[]>(
    query ? `/api/search?q=${query}` : null
  );

  switch (state.status) {
    case 'idle':
      return <p>Enter a search term</p>;
    case 'loading':
      return <ResultsSkeleton />;
    case 'error':
      return <p>Error: {state.error}</p>;
    case 'success':
      return state.data.length === 0
        ? <p>No results found</p>
        : <ProductList products={state.data} />;
  }
}
```


## Resources

- [Fetching data with Effects](https://react.dev/reference/react/useEffect#fetching-data-with-effects) — Official React guide on data fetching patterns

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*