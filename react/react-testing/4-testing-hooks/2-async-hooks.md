---
source_course: "react-testing"
source_lesson: "react-testing-testing-async-hooks"
---

# Testing Async Hooks

Hooks that fetch data or perform async operations need special handling.

## Hook with Data Fetching

```jsx
function useFetchUser(userId) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false;
    
    async function fetchUser() {
      try {
        setLoading(true);
        const response = await fetch(`/api/users/${userId}`);
        const data = await response.json();
        
        if (!cancelled) {
          setUser(data);
          setLoading(false);
        }
      } catch (err) {
        if (!cancelled) {
          setError(err);
          setLoading(false);
        }
      }
    }
    
    fetchUser();
    return () => { cancelled = true; };
  }, [userId]);

  return { user, loading, error };
}
```

## Testing the Hook

```jsx
import { renderHook, waitFor } from '@testing-library/react';
import { server } from '../mocks/server';
import { rest } from 'msw';

test('fetches user data', async () => {
  server.use(
    rest.get('/api/users/:id', (req, res, ctx) => {
      return res(ctx.json({ id: '1', name: 'John' }));
    })
  );
  
  const { result } = renderHook(() => useFetchUser('1'));
  
  // Initially loading
  expect(result.current.loading).toBe(true);
  expect(result.current.user).toBe(null);
  
  // Wait for data
  await waitFor(() => {
    expect(result.current.loading).toBe(false);
  });
  
  expect(result.current.user).toEqual({ id: '1', name: 'John' });
  expect(result.current.error).toBe(null);
});

test('handles fetch error', async () => {
  server.use(
    rest.get('/api/users/:id', (req, res, ctx) => {
      return res(ctx.status(500));
    })
  );
  
  const { result } = renderHook(() => useFetchUser('1'));
  
  await waitFor(() => {
    expect(result.current.loading).toBe(false);
  });
  
  expect(result.current.error).toBeTruthy();
  expect(result.current.user).toBe(null);
});
```

## Async act()

For async operations within the hook:

```jsx
test('async state update', async () => {
  const { result } = renderHook(() => useAsyncCounter());
  
  await act(async () => {
    await result.current.incrementAsync();
  });
  
  expect(result.current.count).toBe(1);
});
```

## Testing Cleanup

```jsx
test('cancels fetch on unmount', async () => {
  const { result, unmount } = renderHook(() => useFetchUser('1'));
  
  // Unmount before fetch completes
  unmount();
  
  // No errors should be thrown
  // State should not be updated after unmount
});
```

---

> 📘 *This lesson is part of the [React Testing Strategies](https://stanza.dev/courses/react-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*