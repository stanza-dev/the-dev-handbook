---
source_course: "react-custom-hooks"
source_lesson: "react-custom-hooks-typescript-hooks"
---

# TypeScript for Custom Hooks

Type-safe custom hooks.

## Basic Typing

```tsx
function useToggle(initialValue: boolean = false) {
  const [value, setValue] = useState(initialValue);
  
  const toggle = useCallback(() => setValue(v => !v), []);
  const setTrue = useCallback(() => setValue(true), []);
  const setFalse = useCallback(() => setValue(false), []);
  
  return [value, { toggle, setTrue, setFalse }] as const;
}

// TypeScript infers: [boolean, { toggle: () => void, ... }]
```

## Generic Hooks

```tsx
function useLocalStorage<T>(key: string, initialValue: T) {
  const [value, setValue] = useState<T>(() => {
    const stored = localStorage.getItem(key);
    return stored ? JSON.parse(stored) : initialValue;
  });
  
  const setStoredValue = useCallback((newValue: T | ((prev: T) => T)) => {
    setValue(prev => {
      const resolved = newValue instanceof Function ? newValue(prev) : newValue;
      localStorage.setItem(key, JSON.stringify(resolved));
      return resolved;
    });
  }, [key]);
  
  return [value, setStoredValue] as const;
}

// Usage - T is inferred
const [user, setUser] = useLocalStorage('user', { name: '', email: '' });
// TypeScript knows: user is { name: string, email: string }
```

## Complex Return Types

```tsx
type FetchState<T> = {
  data: T | null;
  loading: boolean;
  error: Error | null;
};

type FetchReturn<T> = FetchState<T> & {
  refetch: () => Promise<void>;
};

function useFetch<T>(url: string): FetchReturn<T> {
  const [state, setState] = useState<FetchState<T>>({
    data: null,
    loading: true,
    error: null,
  });
  
  const refetch = useCallback(async () => {
    setState(s => ({ ...s, loading: true }));
    try {
      const response = await fetch(url);
      const data = await response.json();
      setState({ data, loading: false, error: null });
    } catch (error) {
      setState({ data: null, loading: false, error: error as Error });
    }
  }, [url]);
  
  useEffect(() => {
    refetch();
  }, [refetch]);
  
  return { ...state, refetch };
}

// Usage
type User = { id: string; name: string };
const { data, loading } = useFetch<User>('/api/user');
// data is User | null
```

## Overloaded Hooks

```tsx
// Different return types based on input
function useMediaQuery(query: string): boolean;
function useMediaQuery(queries: string[]): boolean[];
function useMediaQuery(query: string | string[]): boolean | boolean[] {
  const isArray = Array.isArray(query);
  const queries = isArray ? query : [query];
  
  const [matches, setMatches] = useState(
    queries.map(q => window.matchMedia(q).matches)
  );
  
  useEffect(() => {
    const mediaQueries = queries.map(q => window.matchMedia(q));
    const handlers = mediaQueries.map((mq, i) => {
      const handler = (e: MediaQueryListEvent) => {
        setMatches(prev => {
          const next = [...prev];
          next[i] = e.matches;
          return next;
        });
      };
      mq.addEventListener('change', handler);
      return () => mq.removeEventListener('change', handler);
    });
    
    return () => handlers.forEach(cleanup => cleanup());
  }, [queries.join(',')]);
  
  return isArray ? matches : matches[0];
}

// Usage
const isMobile = useMediaQuery('(max-width: 768px)'); // boolean
const [isMobile, isTablet] = useMediaQuery([
  '(max-width: 768px)',
  '(max-width: 1024px)'
]); // boolean[]
```

## Constrained Generics

```tsx
function useField<T extends string | number | boolean>(
  initialValue: T
) {
  const [value, setValue] = useState<T>(initialValue);
  
  const onChange = useCallback((e: React.ChangeEvent<HTMLInputElement>) => {
    const newValue = (
      typeof initialValue === 'boolean'
        ? e.target.checked
        : e.target.value
    ) as T;
    setValue(newValue);
  }, []);
  
  return { value, onChange };
}
```

---

> 📘 *This lesson is part of the [Building Custom Hooks](https://stanza.dev/courses/react-custom-hooks) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*