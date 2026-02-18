---
source_course: "react-custom-hooks"
source_lesson: "react-custom-hooks-memo-hooks"
---

# usePrevious and useLatest

Access previous values and latest refs.

## usePrevious

```jsx
function usePrevious(value) {
  const ref = useRef();
  
  useEffect(() => {
    ref.current = value;
  }, [value]);
  
  return ref.current;
}

// Usage - Compare with previous
function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);
  
  return (
    <div>
      <p>Current: {count}</p>
      <p>Previous: {prevCount}</p>
      <p>Change: {count - (prevCount ?? 0)}</p>
      <button onClick={() => setCount(c => c + 1)}>+</button>
    </div>
  );
}
```

## usePrevious with Initial Value

```jsx
function usePrevious(value, initialValue) {
  const ref = useRef(initialValue);
  
  useEffect(() => {
    ref.current = value;
  }, [value]);
  
  return ref.current;
}
```

## useLatest

```jsx
function useLatest(value) {
  const ref = useRef(value);
  ref.current = value;
  return ref;
}

// Usage - Always access latest value in callbacks
function Timer({ duration, onComplete }) {
  const latestOnComplete = useLatest(onComplete);
  
  useEffect(() => {
    const timer = setTimeout(() => {
      // Always calls the latest onComplete
      latestOnComplete.current();
    }, duration);
    
    return () => clearTimeout(timer);
  }, [duration]); // No need to include onComplete
}
```

## useWhyDidYouUpdate

```jsx
function useWhyDidYouUpdate(name, props) {
  const previousProps = useRef();
  
  useEffect(() => {
    if (previousProps.current) {
      const allKeys = Object.keys({ ...previousProps.current, ...props });
      const changes = {};
      
      allKeys.forEach(key => {
        if (previousProps.current[key] !== props[key]) {
          changes[key] = {
            from: previousProps.current[key],
            to: props[key],
          };
        }
      });
      
      if (Object.keys(changes).length) {
        console.log('[why-did-you-update]', name, changes);
      }
    }
    
    previousProps.current = props;
  });
}

// Usage - Debug re-renders
function ExpensiveComponent(props) {
  useWhyDidYouUpdate('ExpensiveComponent', props);
  
  return /* ... */;
}
```

## useUpdateEffect

```jsx
function useUpdateEffect(effect, deps) {
  const isFirstMount = useRef(true);
  
  useEffect(() => {
    if (isFirstMount.current) {
      isFirstMount.current = false;
      return;
    }
    
    return effect();
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, deps);
}

// Usage - Skip initial render
function SearchResults({ query }) {
  // Don't fetch on mount, only when query changes
  useUpdateEffect(() => {
    fetchResults(query);
  }, [query]);
  
  return /* ... */;
}
```

## useIsFirstRender

```jsx
function useIsFirstRender() {
  const isFirst = useRef(true);
  
  if (isFirst.current) {
    isFirst.current = false;
    return true;
  }
  
  return false;
}

// Usage
function Component() {
  const isFirstRender = useIsFirstRender();
  
  if (isFirstRender) {
    console.log('First render!');
  }
  
  return /* ... */;
}
```

---

> 📘 *This lesson is part of the [Building Custom Hooks](https://stanza.dev/courses/react-custom-hooks) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*