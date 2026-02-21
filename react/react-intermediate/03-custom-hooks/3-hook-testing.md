---
source_course: "react-intermediate"
source_lesson: "react-custom-hook-testing"
---

# Testing Custom Hooks

## Introduction

Custom hooks are pure logic without a UI, which makes them excellent candidates for testing. Testing hooks ensures they behave correctly across different inputs, handle edge cases properly, and do not break when you refactor their internals. This lesson covers how to test custom hooks using `renderHook` from the React Testing Library.

## Key Concepts

- **renderHook**: A utility from `@testing-library/react` that renders a hook in a test environment without needing a component wrapper.
- **act**: A function that wraps code causing state updates, ensuring React processes the updates before assertions.
- **Rerender**: Simulating prop changes by calling `rerender` with new arguments.
- **Cleanup**: Verifying that hooks clean up properly (clear timers, close connections, abort requests).

## Real World Context

Your team builds a `useAuth` hook that manages login, logout, token refresh, and session expiry. This hook is used across dozens of components. If a refactor introduces a bug (e.g., the token is not refreshed correctly), many features break. Comprehensive hook tests catch these regressions before they reach production.

## Deep Dive

### Setting Up

```tsx
import { renderHook, act } from '@testing-library/react';
```

### Testing a Simple Hook

```tsx
// useCounter.ts
function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);
  const increment = () => setCount(c => c + 1);
  const decrement = () => setCount(c => c - 1);
  const reset = () => setCount(initialValue);
  return { count, increment, decrement, reset };
}

// useCounter.test.ts
describe('useCounter', () => {
  test('starts with initial value', () => {
    const { result } = renderHook(() => useCounter(10));
    expect(result.current.count).toBe(10);
  });

  test('increments the counter', () => {
    const { result } = renderHook(() => useCounter(0));
    act(() => {
      result.current.increment();
    });
    expect(result.current.count).toBe(1);
  });

  test('resets to initial value', () => {
    const { result } = renderHook(() => useCounter(5));
    act(() => {
      result.current.increment();
      result.current.increment();
      result.current.reset();
    });
    expect(result.current.count).toBe(5);
  });
});
```

### Testing Hooks with Dependencies

```tsx
// Testing useDebounce
describe('useDebounce', () => {
  beforeEach(() => jest.useFakeTimers());
  afterEach(() => jest.useRealTimers());

  test('returns initial value immediately', () => {
    const { result } = renderHook(() => useDebounce('hello', 500));
    expect(result.current).toBe('hello');
  });

  test('updates value after delay', () => {
    const { result, rerender } = renderHook(
      ({ value, delay }) => useDebounce(value, delay),
      { initialProps: { value: 'hello', delay: 500 } }
    );

    rerender({ value: 'world', delay: 500 });
    expect(result.current).toBe('hello'); // Not yet updated

    act(() => jest.advanceTimersByTime(500));
    expect(result.current).toBe('world'); // Now updated
  });
});
```

### Testing Async Hooks

```tsx
describe('useFetch', () => {
  test('returns data on successful fetch', async () => {
    global.fetch = jest.fn().mockResolvedValue({
      ok: true,
      json: () => Promise.resolve({ name: 'Alice' }),
    });

    const { result } = renderHook(() => useFetch('/api/user'));

    expect(result.current.loading).toBe(true);

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
      expect(result.current.data).toEqual({ name: 'Alice' });
    });
  });

  test('returns error on failed fetch', async () => {
    global.fetch = jest.fn().mockRejectedValue(new Error('Network error'));

    const { result } = renderHook(() => useFetch('/api/user'));

    await waitFor(() => {
      expect(result.current.error?.message).toBe('Network error');
    });
  });
});
```

## Common Pitfalls

1. **Forgetting to wrap state updates in act()** — React warns when state updates happen outside of `act()`. Always wrap calls to hook functions that trigger state changes.
2. **Not waiting for async effects** — Use `waitFor` from Testing Library to wait for async state changes rather than relying on fixed timeouts.

## Best Practices

1. **Test behavior, not implementation** — Assert on the hook's return values and side effects, not on internal state variables or how many times a function was called.
2. **Test edge cases** — Empty inputs, rapid successive calls, unmounting during async operations, and error conditions are where bugs hide.

## Summary

- Use `renderHook` from React Testing Library to test custom hooks in isolation.
- Wrap state-triggering actions in `act()` and use `waitFor` for async operations.
- Test behavior (return values and side effects) rather than internal implementation details.

## Code Examples

**Testing a useToggle custom hook with renderHook and act**

```tsx
import { renderHook, act } from '@testing-library/react';

function useToggle(initial = false) {
  const [value, setValue] = useState(initial);
  const toggle = useCallback(() => setValue(v => !v), []);
  const setTrue = useCallback(() => setValue(true), []);
  const setFalse = useCallback(() => setValue(false), []);
  return { value, toggle, setTrue, setFalse };
}

describe('useToggle', () => {
  test('toggles between true and false', () => {
    const { result } = renderHook(() => useToggle());
    expect(result.current.value).toBe(false);

    act(() => result.current.toggle());
    expect(result.current.value).toBe(true);

    act(() => result.current.toggle());
    expect(result.current.value).toBe(false);
  });
});
```


## Resources

- [Testing Library - renderHook](https://testing-library.com/docs/react-testing-library/api#renderhook) — Official renderHook API reference

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*