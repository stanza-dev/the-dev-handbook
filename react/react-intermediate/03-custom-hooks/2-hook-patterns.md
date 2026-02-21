---
source_course: "react-intermediate"
source_lesson: "react-custom-hook-patterns"
---

# Custom Hook Patterns

## Introduction

Once you understand the basics of custom hooks, you can build increasingly powerful abstractions. This lesson covers advanced patterns: hooks that wrap async operations, hooks that manage complex state machines, and hooks that compose other hooks. These patterns are the building blocks of production-grade React applications.

## Key Concepts

- **Async Hook Pattern**: A hook that manages loading, data, and error states for asynchronous operations.
- **State Machine Hook**: A hook that encapsulates a finite state machine with defined transitions.
- **Composed Hooks**: Building complex hooks by combining simpler hooks.
- **Generic Hooks**: Using TypeScript generics to create type-safe, reusable hooks.

## Real World Context

Every production application needs to fetch data, handle loading states, manage errors, and often retry failed requests. Libraries like TanStack Query started as custom hooks. Understanding the async hook pattern helps you both use these libraries effectively and build custom solutions when a library is overkill.

## Deep Dive

### The Async Hook Pattern

```tsx
type AsyncState<T> =
  | { status: 'idle'; data: null; error: null }
  | { status: 'loading'; data: null; error: null }
  | { status: 'success'; data: T; error: null }
  | { status: 'error'; data: null; error: Error };

function useAsync<T>(asyncFn: () => Promise<T>, deps: unknown[]) {
  const [state, setState] = useState<AsyncState<T>>({
    status: 'idle',
    data: null,
    error: null,
  });

  useEffect(() => {
    let cancelled = false;
    setState({ status: 'loading', data: null, error: null });

    asyncFn()
      .then(data => {
        if (!cancelled) setState({ status: 'success', data, error: null });
      })
      .catch(error => {
        if (!cancelled) setState({ status: 'error', data: null, error });
      });

    return () => { cancelled = true; };
  }, deps);

  return state;
}

// Usage
function UserProfile({ userId }: { userId: string }) {
  const { status, data: user, error } = useAsync(
    () => fetch(`/api/users/${userId}`).then(r => r.json()),
    [userId]
  );

  if (status === 'loading') return <Spinner />;
  if (status === 'error') return <p>Error: {error.message}</p>;
  if (status === 'idle' || !user) return null;

  return <h1>{user.name}</h1>;
}
```

### Composing Hooks

Build complex hooks from simpler ones:

```tsx
function useSearchWithDebounce(apiUrl: string) {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebounce(query, 300);

  const { data: results, status } = useAsync(
    () => debouncedQuery
      ? fetch(`${apiUrl}?q=${debouncedQuery}`).then(r => r.json())
      : Promise.resolve([]),
    [debouncedQuery, apiUrl]
  );

  return { query, setQuery, results: results || [], isSearching: status === 'loading' };
}
```

### The Reducer Hook Pattern

For complex state with many transitions, combine `useReducer` with a custom hook:

```tsx
type WizardState = {
  step: number;
  data: Record<string, unknown>;
  isComplete: boolean;
};

type WizardAction =
  | { type: 'next'; payload: Record<string, unknown> }
  | { type: 'back' }
  | { type: 'complete' }
  | { type: 'reset' };

function useWizard(totalSteps: number) {
  const [state, dispatch] = useReducer(
    (state: WizardState, action: WizardAction): WizardState => {
      switch (action.type) {
        case 'next':
          return {
            ...state,
            step: Math.min(state.step + 1, totalSteps - 1),
            data: { ...state.data, ...action.payload },
          };
        case 'back':
          return { ...state, step: Math.max(state.step - 1, 0) };
        case 'complete':
          return { ...state, isComplete: true };
        case 'reset':
          return { step: 0, data: {}, isComplete: false };
        default:
          return state;
      }
    },
    { step: 0, data: {}, isComplete: false }
  );

  return {
    step: state.step,
    data: state.data,
    isComplete: state.isComplete,
    isFirst: state.step === 0,
    isLast: state.step === totalSteps - 1,
    next: (data: Record<string, unknown>) => dispatch({ type: 'next', payload: data }),
    back: () => dispatch({ type: 'back' }),
    complete: () => dispatch({ type: 'complete' }),
    reset: () => dispatch({ type: 'reset' }),
  };
}
```

## Common Pitfalls

1. **Unstable dependency arrays in composed hooks** — When passing functions or objects as hook arguments, make sure they are stable (memoized) to avoid infinite re-render loops.
2. **Missing cancellation in async hooks** — Always use a `cancelled` flag or `AbortController` to prevent setting state on unmounted components.

## Best Practices

1. **Type the return value** — Use TypeScript discriminated unions for async state so consumers can narrow types with status checks.
2. **Accept configuration over hard-coding behavior** — Make hooks flexible by accepting options: `useDebounce(value, delay)` is more reusable than `useDebounce300ms(value)`.

## Summary

- The async hook pattern encapsulates loading, success, and error states in a reusable way.
- Compose complex hooks from simpler ones: debounce + async = search with debounce.
- Use TypeScript discriminated unions and generics to make hooks type-safe and flexible.

## Code Examples

**A type-safe useFetch hook with AbortController cleanup**

```tsx
function useFetch<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    const controller = new AbortController();
    setLoading(true);

    fetch(url, { signal: controller.signal })
      .then(res => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        return res.json();
      })
      .then(json => { setData(json); setError(null); })
      .catch(err => {
        if (err.name !== 'AbortError') setError(err);
      })
      .finally(() => setLoading(false));

    return () => controller.abort();
  }, [url]);

  return { data, loading, error };
}
```


## Resources

- [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks) — Official guide on custom hook patterns

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*