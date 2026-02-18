---
source_course: "react-testing"
source_lesson: "react-testing-render-hook-basics"
---

# renderHook Basics

Custom hooks can't be called outside components. `renderHook` solves this!

## Basic Usage

```jsx
import { renderHook, act } from '@testing-library/react';
import useCounter from './useCounter';

test('initializes with default value', () => {
  const { result } = renderHook(() => useCounter());
  
  expect(result.current.count).toBe(0);
});

test('initializes with custom value', () => {
  const { result } = renderHook(() => useCounter(10));
  
  expect(result.current.count).toBe(10);
});
```

## The result Object

`result.current` always has the latest return value:

```jsx
function useCounter(initial = 0) {
  const [count, setCount] = useState(initial);
  
  const increment = () => setCount(c => c + 1);
  const decrement = () => setCount(c => c - 1);
  const reset = () => setCount(initial);
  
  return { count, increment, decrement, reset };
}

test('increment increases count', () => {
  const { result } = renderHook(() => useCounter());
  
  act(() => {
    result.current.increment();
  });
  
  expect(result.current.count).toBe(1);
});
```

## Using act()

Wrap state-changing calls in `act()`:

```jsx
test('multiple operations', () => {
  const { result } = renderHook(() => useCounter(5));
  
  act(() => {
    result.current.increment();
    result.current.increment();
  });
  
  expect(result.current.count).toBe(7);
  
  act(() => {
    result.current.reset();
  });
  
  expect(result.current.count).toBe(5);
});
```

## Testing with Props/Arguments

Use `initialProps` and `rerender`:

```jsx
function useGreeting(name) {
  return `Hello, ${name}!`;
}

test('updates when prop changes', () => {
  const { result, rerender } = renderHook(
    ({ name }) => useGreeting(name),
    { initialProps: { name: 'Alice' } }
  );
  
  expect(result.current).toBe('Hello, Alice!');
  
  rerender({ name: 'Bob' });
  
  expect(result.current).toBe('Hello, Bob!');
});
```

## What renderHook Returns

```jsx
const {
  result,    // { current: <hook return value> }
  rerender,  // (newProps) => void
  unmount    // () => void
} = renderHook(hook, options);
```

---

> 📘 *This lesson is part of the [React Testing Strategies](https://stanza.dev/courses/react-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*