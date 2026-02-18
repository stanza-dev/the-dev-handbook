---
source_course: "react-hooks-deep-dive"
source_lesson: "react-hooks-deep-dive-custom-hooks"
---

# Custom Hooks: Reusable Logic

Custom hooks let you extract component logic into reusable functions. They're just JavaScript functions that use other hooks.

## Naming Convention

Custom hooks must start with `use` followed by a capital letter:

```tsx
// ✅ Valid custom hook names
useAuth
useLocalStorage
useFetch
useWindowSize

// ❌ Invalid - won't be recognized as hooks
getAuth
fetchData
withAuth
```

## Example: useLocalStorage

```tsx
function useLocalStorage<T>(
  key: string,
  initialValue: T
): [T, (value: T | ((prev: T) => T)) => void] {
  // Get initial value from storage or use default
  const [storedValue, setStoredValue] = useState<T>(() => {
    if (typeof window === 'undefined') return initialValue;
    
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });

  // Update storage when value changes
  const setValue = useCallback((value: T | ((prev: T) => T)) => {
    setStoredValue((prev) => {
      const newValue = value instanceof Function ? value(prev) : value;
      
      if (typeof window !== 'undefined') {
        window.localStorage.setItem(key, JSON.stringify(newValue));
      }
      
      return newValue;
    });
  }, [key]);

  return [storedValue, setValue];
}

// Usage
function Settings() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  // ...
}
```

## Example: useFetch

```tsx
type FetchState<T> = {
  data: T | null;
  isLoading: boolean;
  error: Error | null;
};

function useFetch<T>(url: string): FetchState<T> {
  const [state, setState] = useState<FetchState<T>>({
    data: null,
    isLoading: true,
    error: null
  });

  useEffect(() => {
    let cancelled = false;

    setState({ data: null, isLoading: true, error: null });

    fetch(url)
      .then((res) => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        return res.json();
      })
      .then((data) => {
        if (!cancelled) {
          setState({ data, isLoading: false, error: null });
        }
      })
      .catch((error) => {
        if (!cancelled) {
          setState({ data: null, isLoading: false, error });
        }
      });

    return () => {
      cancelled = true;
    };
  }, [url]);

  return state;
}
```

## Best Practices

1. **Single responsibility**: Each hook should do one thing well
2. **Return what's needed**: Don't expose internal implementation details
3. **Handle cleanup**: Always clean up subscriptions and async operations
4. **Type properly**: Use TypeScript generics for flexibility
5. **Document behavior**: Especially edge cases and requirements

## Resources

- [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks) — Official React guide on building custom hooks

---

> 📘 *This lesson is part of the [React Hooks Deep Dive](https://stanza.dev/courses/react-hooks-deep-dive) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*