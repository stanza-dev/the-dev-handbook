---
source_course: "react-custom-hooks"
source_lesson: "react-custom-hooks-intersection-observer"
---

# useIntersectionObserver

Detect when elements enter the viewport.

## Basic Implementation

```jsx
function useIntersectionObserver(options = {}) {
  const [ref, setRef] = useState(null);
  const [entry, setEntry] = useState(null);
  
  useEffect(() => {
    if (!ref) return;
    
    const observer = new IntersectionObserver(
      ([entry]) => setEntry(entry),
      options
    );
    
    observer.observe(ref);
    
    return () => observer.disconnect();
  }, [ref, options.threshold, options.root, options.rootMargin]);
  
  return [setRef, entry];
}

// Usage - Lazy load images
function LazyImage({ src, alt }) {
  const [ref, entry] = useIntersectionObserver({
    rootMargin: '100px',
  });
  const [loaded, setLoaded] = useState(false);
  
  useEffect(() => {
    if (entry?.isIntersecting) {
      setLoaded(true);
    }
  }, [entry]);
  
  return (
    <div ref={ref}>
      {loaded ? (
        <img src={src} alt={alt} />
      ) : (
        <div className="placeholder" />
      )}
    </div>
  );
}
```

## useOnScreen

```jsx
function useOnScreen(options = {}) {
  const [ref, setRef] = useState(null);
  const [isOnScreen, setIsOnScreen] = useState(false);
  
  useEffect(() => {
    if (!ref) return;
    
    const observer = new IntersectionObserver(
      ([entry]) => setIsOnScreen(entry.isIntersecting),
      options
    );
    
    observer.observe(ref);
    return () => observer.disconnect();
  }, [ref, options]);
  
  return [setRef, isOnScreen];
}

// Usage - Animate on scroll
function AnimatedSection({ children }) {
  const [ref, isVisible] = useOnScreen({ threshold: 0.1 });
  
  return (
    <div
      ref={ref}
      className={`section ${isVisible ? 'visible' : ''}`}
    >
      {children}
    </div>
  );
}
```

## Infinite Scroll Hook

```jsx
function useInfiniteScroll(callback, options = {}) {
  const [ref, setRef] = useState(null);
  
  useEffect(() => {
    if (!ref) return;
    
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          callback();
        }
      },
      options
    );
    
    observer.observe(ref);
    return () => observer.disconnect();
  }, [ref, callback]);
  
  return setRef;
}

// Usage
function InfiniteList() {
  const [items, setItems] = useState(initialItems);
  const [page, setPage] = useState(1);
  const [loading, setLoading] = useState(false);
  
  const loadMore = useCallback(async () => {
    if (loading) return;
    setLoading(true);
    const newItems = await fetchItems(page + 1);
    setItems(prev => [...prev, ...newItems]);
    setPage(p => p + 1);
    setLoading(false);
  }, [page, loading]);
  
  const sentinelRef = useInfiniteScroll(loadMore);
  
  return (
    <div>
      {items.map(item => (
        <Item key={item.id} {...item} />
      ))}
      <div ref={sentinelRef} className="sentinel">
        {loading && <Spinner />}
      </div>
    </div>
  );
}
```

## Track Multiple Elements

```jsx
function useIntersectionObserverMultiple(options = {}) {
  const [entries, setEntries] = useState(new Map());
  const observerRef = useRef(null);
  
  useEffect(() => {
    observerRef.current = new IntersectionObserver(
      (entries) => {
        setEntries(prev => {
          const next = new Map(prev);
          entries.forEach(entry => {
            next.set(entry.target, entry);
          });
          return next;
        });
      },
      options
    );
    
    return () => observerRef.current?.disconnect();
  }, [options]);
  
  const observe = useCallback((element) => {
    if (element && observerRef.current) {
      observerRef.current.observe(element);
    }
  }, []);
  
  const unobserve = useCallback((element) => {
    if (element && observerRef.current) {
      observerRef.current.unobserve(element);
    }
  }, []);
  
  return { entries, observe, unobserve };
}
```

---

> 📘 *This lesson is part of the [Building Custom Hooks](https://stanza.dev/courses/react-custom-hooks) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*