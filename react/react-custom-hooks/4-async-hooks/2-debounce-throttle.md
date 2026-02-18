---
source_course: "react-custom-hooks"
source_lesson: "react-custom-hooks-debounce-throttle"
---

# useDebounce and useThrottle

Control the rate of value changes.

## useDebounce

```jsx
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);
  
  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    return () => clearTimeout(timer);
  }, [value, delay]);
  
  return debouncedValue;
}

// Usage - Search
function Search() {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebounce(query, 300);
  
  const { data: results } = useQuery(
    ['search', debouncedQuery],
    () => searchAPI(debouncedQuery),
    { enabled: debouncedQuery.length > 0 }
  );
  
  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search..."
      />
      <SearchResults results={results} />
    </div>
  );
}
```

## useDebouncedCallback

```jsx
function useDebouncedCallback(callback, delay) {
  const callbackRef = useRef(callback);
  callbackRef.current = callback;
  
  const timeoutRef = useRef();
  
  const debouncedCallback = useCallback((...args) => {
    clearTimeout(timeoutRef.current);
    timeoutRef.current = setTimeout(() => {
      callbackRef.current(...args);
    }, delay);
  }, [delay]);
  
  // Cancel on unmount
  useEffect(() => {
    return () => clearTimeout(timeoutRef.current);
  }, []);
  
  return debouncedCallback;
}

// Usage
function AutoSave() {
  const [content, setContent] = useState('');
  
  const save = useDebouncedCallback((text) => {
    saveToServer(text);
  }, 1000);
  
  const handleChange = (e) => {
    setContent(e.target.value);
    save(e.target.value);
  };
  
  return <textarea value={content} onChange={handleChange} />;
}
```

## useThrottle

```jsx
function useThrottle(value, interval) {
  const [throttledValue, setThrottledValue] = useState(value);
  const lastUpdated = useRef(Date.now());
  
  useEffect(() => {
    const now = Date.now();
    
    if (now >= lastUpdated.current + interval) {
      lastUpdated.current = now;
      setThrottledValue(value);
    } else {
      const timer = setTimeout(() => {
        lastUpdated.current = Date.now();
        setThrottledValue(value);
      }, interval - (now - lastUpdated.current));
      
      return () => clearTimeout(timer);
    }
  }, [value, interval]);
  
  return throttledValue;
}

// Usage - Scroll position
function ScrollTracker() {
  const [scrollY, setScrollY] = useState(0);
  const throttledScrollY = useThrottle(scrollY, 100);
  
  useEffect(() => {
    const handleScroll = () => setScrollY(window.scrollY);
    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, []);
  
  // Only updates every 100ms
  return <ScrollIndicator position={throttledScrollY} />;
}
```

## useThrottledCallback

```jsx
function useThrottledCallback(callback, limit) {
  const lastRan = useRef(Date.now());
  const lastCallback = useRef();
  
  return useCallback((...args) => {
    const now = Date.now();
    
    if (now - lastRan.current >= limit) {
      callback(...args);
      lastRan.current = now;
    } else {
      clearTimeout(lastCallback.current);
      lastCallback.current = setTimeout(() => {
        callback(...args);
        lastRan.current = Date.now();
      }, limit - (now - lastRan.current));
    }
  }, [callback, limit]);
}

// Usage - Resize handler
function ResizableComponent() {
  const handleResize = useThrottledCallback((width, height) => {
    expensiveCalculation(width, height);
  }, 200);
  
  // Called frequently, but runs at most every 200ms
  return <Resizable onResize={handleResize} />;
}
```

## Debounce vs Throttle

| | Debounce | Throttle |
|---|---|---|
| **When** | After pause in events | At regular intervals |
| **Use for** | Search input, resize end | Scroll, mousemove |
| **Behavior** | Waits for silence | Limits frequency |

---

> 📘 *This lesson is part of the [Building Custom Hooks](https://stanza.dev/courses/react-custom-hooks) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*