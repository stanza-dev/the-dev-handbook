---
source_course: "react-custom-hooks"
source_lesson: "react-custom-hooks-testing-hooks"
---

# Testing Custom Hooks

Use renderHook from React Testing Library.

## Setup

```bash
npm install -D @testing-library/react
```

## Basic Hook Test

```jsx
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

test('useCounter increments', () => {
  const { result } = renderHook(() => useCounter());
  
  expect(result.current.count).toBe(0);
  
  act(() => {
    result.current.increment();
  });
  
  expect(result.current.count).toBe(1);
});

test('useCounter with initial value', () => {
  const { result } = renderHook(() => useCounter(10));
  
  expect(result.current.count).toBe(10);
});
```

## Testing Hook with Props/Dependencies

```jsx
import { renderHook } from '@testing-library/react';
import { useFetch } from './useFetch';

test('useFetch re-fetches when URL changes', async () => {
  const { result, rerender } = renderHook(
    ({ url }) => useFetch(url),
    { initialProps: { url: '/api/users' } }
  );
  
  expect(result.current.loading).toBe(true);
  
  // Wait for fetch to complete
  await waitFor(() => {
    expect(result.current.loading).toBe(false);
  });
  
  // Change URL
  rerender({ url: '/api/posts' });
  
  expect(result.current.loading).toBe(true);
});
```

## Testing Async Hooks

```jsx
import { renderHook, waitFor } from '@testing-library/react';
import { useAsync } from './useAsync';

const mockFetch = jest.fn();

test('useAsync handles success', async () => {
  mockFetch.mockResolvedValueOnce({ data: 'test' });
  
  const { result } = renderHook(() => 
    useAsync(() => mockFetch())
  );
  
  expect(result.current.status).toBe('pending');
  
  await waitFor(() => {
    expect(result.current.status).toBe('success');
  });
  
  expect(result.current.value).toEqual({ data: 'test' });
});

test('useAsync handles error', async () => {
  mockFetch.mockRejectedValueOnce(new Error('Failed'));
  
  const { result } = renderHook(() => 
    useAsync(() => mockFetch())
  );
  
  await waitFor(() => {
    expect(result.current.status).toBe('error');
  });
  
  expect(result.current.error.message).toBe('Failed');
});
```

## Testing with Context

```jsx
import { renderHook } from '@testing-library/react';
import { useAuth } from './useAuth';
import { AuthProvider } from './AuthContext';

test('useAuth returns user from context', () => {
  const wrapper = ({ children }) => (
    <AuthProvider value={{ user: { name: 'John' } }}>
      {children}
    </AuthProvider>
  );
  
  const { result } = renderHook(() => useAuth(), { wrapper });
  
  expect(result.current.user.name).toBe('John');
});
```

## Testing Cleanup

```jsx
test('useEventListener cleans up', () => {
  const handler = jest.fn();
  const addSpy = jest.spyOn(window, 'addEventListener');
  const removeSpy = jest.spyOn(window, 'removeEventListener');
  
  const { unmount } = renderHook(() => 
    useEventListener('resize', handler)
  );
  
  expect(addSpy).toHaveBeenCalledWith('resize', expect.any(Function));
  
  unmount();
  
  expect(removeSpy).toHaveBeenCalledWith('resize', expect.any(Function));
});
```

## Testing Timer-Based Hooks

```jsx
import { renderHook, act } from '@testing-library/react';
import { useDebounce } from './useDebounce';

test('useDebounce debounces value', () => {
  jest.useFakeTimers();
  
  const { result, rerender } = renderHook(
    ({ value }) => useDebounce(value, 500),
    { initialProps: { value: 'initial' } }
  );
  
  expect(result.current).toBe('initial');
  
  rerender({ value: 'updated' });
  
  // Value hasn't changed yet
  expect(result.current).toBe('initial');
  
  // Fast-forward time
  act(() => {
    jest.advanceTimersByTime(500);
  });
  
  expect(result.current).toBe('updated');
  
  jest.useRealTimers();
});
```

---

> 📘 *This lesson is part of the [Building Custom Hooks](https://stanza.dev/courses/react-custom-hooks) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*