---
source_course: "react-performance"
source_lesson: "react-performance-unnecessary-rerenders"
---

# Identifying Unnecessary Re-renders

Find and fix components that render too often.

## React DevTools Profiler

1. Install React DevTools browser extension
2. Open DevTools → Profiler tab
3. Click "Record"
4. Interact with your app
5. Stop recording
6. Analyze the flame graph

### Reading the Flame Graph

- **Gray bars**: Didn't render
- **Blue/green bars**: Rendered (darker = slower)
- **"Why did this render?"**: Click a component to see

## Common Causes of Unnecessary Re-renders

### 1. Inline Object/Array Props

```jsx
// ❌ Bad: New object every render
function Parent() {
  return <Child style={{ color: 'red' }} />;
}

// ✅ Good: Stable reference
const style = { color: 'red' };
function Parent() {
  return <Child style={style} />;
}

// ✅ Or memoize inside component
function Parent() {
  const style = useMemo(() => ({ color: 'red' }), []);
  return <Child style={style} />;
}
```

### 2. Inline Functions

```jsx
// ❌ Bad: New function every render
function Parent() {
  return <Child onClick={() => console.log('clicked')} />;
}

// ✅ Good: Stable callback
function Parent() {
  const handleClick = useCallback(() => {
    console.log('clicked');
  }, []);
  
  return <Child onClick={handleClick} />;
}
```

### 3. Context Changes

```jsx
// ❌ Bad: Context value changes every render
function Provider({ children }) {
  const [user, setUser] = useState(null);
  
  return (
    <Context.Provider value={{ user, setUser }}>
      {children}
    </Context.Provider>
  );
}

// ✅ Good: Memoized value
function Provider({ children }) {
  const [user, setUser] = useState(null);
  const value = useMemo(() => ({ user, setUser }), [user]);
  
  return (
    <Context.Provider value={value}>
      {children}
    </Context.Provider>
  );
}
```

### 4. Parent State Updates

```jsx
// Child re-renders when Parent's count changes
function Parent() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>
        {count}
      </button>
      <ExpensiveChild />  {/* Re-renders every click! */}
    </div>
  );
}

// ✅ Solution: memo
const ExpensiveChild = memo(function ExpensiveChild() {
  // Only re-renders if props change
  return <div>Expensive computation...</div>;
});
```

## Quick Debugging

```jsx
// Add to any component
function DebugRenders({ name }) {
  useEffect(() => {
    console.log(`${name} rendered`);
  });
  return null;
}

function MyComponent() {
  return (
    <div>
      <DebugRenders name="MyComponent" />
      {/* rest of component */}
    </div>
  );
}
```

---

> 📘 *This lesson is part of the [React Performance Deep Dive](https://stanza.dev/courses/react-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*