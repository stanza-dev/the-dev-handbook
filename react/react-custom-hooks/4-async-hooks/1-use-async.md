---
source_course: "react-custom-hooks"
source_lesson: "react-custom-hooks-use-async"
---

# useAsync Hook

Manage async operations with proper states.

## Basic useAsync

```jsx
function useAsync(asyncFunction, immediate = true) {
  const [state, setState] = useState({
    status: 'idle',
    value: null,
    error: null,
  });
  
  const execute = useCallback(async (...args) => {
    setState({ status: 'pending', value: null, error: null });
    try {
      const response = await asyncFunction(...args);
      setState({ status: 'success', value: response, error: null });
      return response;
    } catch (error) {
      setState({ status: 'error', value: null, error });
      throw error;
    }
  }, [asyncFunction]);
  
  useEffect(() => {
    if (immediate) {
      execute();
    }
  }, [execute, immediate]);
  
  return {
    execute,
    ...state,
    isIdle: state.status === 'idle',
    isPending: state.status === 'pending',
    isSuccess: state.status === 'success',
    isError: state.status === 'error',
  };
}

// Usage
function UserProfile({ userId }) {
  const { value: user, isPending, isError, error } = useAsync(
    () => fetchUser(userId),
    true // immediate
  );
  
  if (isPending) return <Spinner />;
  if (isError) return <Error message={error.message} />;
  if (!user) return null;
  
  return <Profile user={user} />;
}
```

## With Dependencies

```jsx
function useAsync(asyncFunction, dependencies = []) {
  const [state, setState] = useState({
    status: 'idle',
    value: null,
    error: null,
  });
  
  const execute = useCallback(async () => {
    setState(s => ({ ...s, status: 'pending' }));
    try {
      const response = await asyncFunction();
      setState({ status: 'success', value: response, error: null });
      return response;
    } catch (error) {
      setState({ status: 'error', value: null, error });
      throw error;
    }
  }, [asyncFunction]);
  
  useEffect(() => {
    execute();
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, dependencies);
  
  return { execute, ...state };
}

// Usage with dependencies
function Posts({ category }) {
  const { value: posts, isPending } = useAsync(
    () => fetchPosts(category),
    [category] // Re-fetch when category changes
  );
  
  return /* ... */;
}
```

## With Cancellation

```jsx
function useAsync(asyncFunction, dependencies = []) {
  const [state, setState] = useState({
    status: 'idle',
    value: null,
    error: null,
  });
  
  useEffect(() => {
    let cancelled = false;
    const controller = new AbortController();
    
    const execute = async () => {
      setState(s => ({ ...s, status: 'pending' }));
      try {
        const response = await asyncFunction(controller.signal);
        if (!cancelled) {
          setState({ status: 'success', value: response, error: null });
        }
      } catch (error) {
        if (!cancelled && error.name !== 'AbortError') {
          setState({ status: 'error', value: null, error });
        }
      }
    };
    
    execute();
    
    return () => {
      cancelled = true;
      controller.abort();
    };
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, dependencies);
  
  return state;
}

// Usage
function SearchResults({ query }) {
  const { value: results, status } = useAsync(
    (signal) => search(query, { signal }),
    [query]
  );
  
  return /* ... */;
}
```

## useAsyncCallback (Manual Trigger)

```jsx
function useAsyncCallback(asyncFunction) {
  const [state, setState] = useState({
    status: 'idle',
    value: null,
    error: null,
  });
  
  const execute = useCallback(async (...args) => {
    setState({ status: 'pending', value: null, error: null });
    try {
      const result = await asyncFunction(...args);
      setState({ status: 'success', value: result, error: null });
      return result;
    } catch (error) {
      setState({ status: 'error', value: null, error });
      throw error;
    }
  }, [asyncFunction]);
  
  const reset = useCallback(() => {
    setState({ status: 'idle', value: null, error: null });
  }, []);
  
  return { execute, reset, ...state };
}

// Usage for form submission
function ContactForm() {
  const { execute: submit, isPending, isSuccess, error } = 
    useAsyncCallback(submitForm);
  
  const handleSubmit = async (data) => {
    await submit(data);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* fields */}
      <button disabled={isPending}>
        {isPending ? 'Sending...' : 'Send'}
      </button>
      {isSuccess && <p>Sent!</p>}
      {error && <p>Error: {error.message}</p>}
    </form>
  );
}
```

---

> 📘 *This lesson is part of the [Building Custom Hooks](https://stanza.dev/courses/react-custom-hooks) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*