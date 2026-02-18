---
source_course: "react-performance"
source_lesson: "react-performance-memory-leaks"
---

# Identifying Memory Leaks

Memory leaks cause performance degradation over time.

## Common Memory Leak Sources

### 1. Uncleared Timers

```jsx
// ❌ Timer not cleared
function Timer() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    setInterval(() => {
      setCount(c => c + 1);
    }, 1000);
    // Missing cleanup!
  }, []);
}

// ✅ Timer cleared on unmount
function Timer() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const intervalId = setInterval(() => {
      setCount(c => c + 1);
    }, 1000);
    
    return () => clearInterval(intervalId);
  }, []);
}
```

### 2. Event Listeners

```jsx
// ❌ Listener not removed
function WindowResize() {
  const [width, setWidth] = useState(window.innerWidth);
  
  useEffect(() => {
    window.addEventListener('resize', () => {
      setWidth(window.innerWidth);
    });
    // Missing cleanup!
  }, []);
}

// ✅ Listener removed on unmount
function WindowResize() {
  const [width, setWidth] = useState(window.innerWidth);
  
  useEffect(() => {
    const handleResize = () => setWidth(window.innerWidth);
    window.addEventListener('resize', handleResize);
    
    return () => window.removeEventListener('resize', handleResize);
  }, []);
}
```

### 3. Subscriptions

```jsx
// ❌ Subscription not unsubscribed
function DataStream() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    const subscription = dataSource.subscribe(setData);
    // Missing cleanup!
  }, []);
}

// ✅ Subscription cleaned up
function DataStream() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    const subscription = dataSource.subscribe(setData);
    
    return () => subscription.unsubscribe();
  }, []);
}
```

### 4. Async Operations After Unmount

```jsx
// ❌ Setting state after unmount
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetchUser(userId).then(setUser);
    // Component might unmount before fetch completes!
  }, [userId]);
}

// ✅ Using AbortController
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    const controller = new AbortController();
    
    fetchUser(userId, { signal: controller.signal })
      .then(setUser)
      .catch(err => {
        if (err.name !== 'AbortError') {
          console.error(err);
        }
      });
    
    return () => controller.abort();
  }, [userId]);
}
```

## Detecting Memory Leaks

### Chrome DevTools

1. Open DevTools → Memory tab
2. Take heap snapshot
3. Use app for a while
4. Take another snapshot
5. Compare snapshots

### Performance Monitor

1. DevTools → More tools → Performance monitor
2. Watch "JS heap size"
3. If it keeps growing, you have a leak

### React DevTools

Look for:
- Mounted components that should be unmounted
- Increasing number of component instances

---

> 📘 *This lesson is part of the [React Performance Deep Dive](https://stanza.dev/courses/react-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*