---
source_course: "react-hooks-deep-dive"
source_lesson: "react-hooks-deep-dive-deep-useref"
---

# useRef: The Escape Hatch

`useRef` returns a mutable ref object whose `.current` property is initialized to the passed argument. The returned object persists for the full lifetime of the component.

## Two Primary Use Cases

### 1. Accessing DOM Elements

```tsx
function TextInput() {
  const inputRef = useRef<HTMLInputElement>(null);

  const focusInput = () => {
    inputRef.current?.focus();
  };

  return (
    <>
      <input ref={inputRef} type="text" />
      <button onClick={focusInput}>Focus Input</button>
    </>
  );
}
```

### 2. Storing Mutable Values Without Re-renders

```tsx
function Timer() {
  const [count, setCount] = useState(0);
  const intervalRef = useRef<number | null>(null);

  const startTimer = () => {
    if (intervalRef.current !== null) return; // Already running
    
    intervalRef.current = window.setInterval(() => {
      setCount((c) => c + 1);
    }, 1000);
  };

  const stopTimer = () => {
    if (intervalRef.current !== null) {
      clearInterval(intervalRef.current);
      intervalRef.current = null;
    }
  };

  // Cleanup on unmount
  useEffect(() => {
    return () => stopTimer();
  }, []);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={startTimer}>Start</button>
      <button onClick={stopTimer}>Stop</button>
    </div>
  );
}
```

## Key Characteristics

| Feature | useRef | useState |
|---------|--------|----------|
| Triggers re-render | ❌ No | ✅ Yes |
| Value persists across renders | ✅ Yes | ✅ Yes |
| Can hold any value | ✅ Yes | ✅ Yes |
| Synchronous updates | ✅ Yes | ❌ No (batched) |

## Common Patterns

### Tracking Previous Values

```tsx
function usePrevious<T>(value: T): T | undefined {
  const ref = useRef<T>();
  
  useEffect(() => {
    ref.current = value;
  });
  
  return ref.current;
}

// Usage
function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);
  
  return (
    <p>
      Current: {count}, Previous: {prevCount}
    </p>
  );
}
```

### Stable Callback Reference

```tsx
function useEventCallback<T extends (...args: any[]) => any>(fn: T): T {
  const ref = useRef(fn);
  
  useEffect(() => {
    ref.current = fn;
  });
  
  return useCallback((...args: Parameters<T>) => {
    return ref.current(...args);
  }, []) as T;
}
```

## Resources

- [useRef API Reference](https://react.dev/reference/react/useRef) — Official React documentation for useRef hook
- [Referencing Values with Refs](https://react.dev/learn/referencing-values-with-refs) — Learn guide on when and how to use refs

---

> 📘 *This lesson is part of the [React Hooks Deep Dive](https://stanza.dev/courses/react-hooks-deep-dive) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*