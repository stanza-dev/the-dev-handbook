---
source_course: "react-performance"
source_lesson: "react-performance-render-phases"
---

# React Render Phases

Understanding React's rendering process is key to optimization.

## The Three Phases

### 1. Trigger Phase
A render is triggered by:
- Initial mount
- State update (`useState`, `useReducer`)
- Parent re-render
- Context value change

### 2. Render Phase
React calls your components to figure out what changed:

```jsx
function Component() {
  // This code runs during render
  console.log('Rendering!');
  
  return <div>Hello</div>;
}
```

**Important**: Render phase should be pure - no side effects!

### 3. Commit Phase
React applies changes to the DOM:
- Updates the actual DOM
- Runs `useLayoutEffect`
- Runs `useEffect` (after paint)

## When Components Re-render

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  
  // When count changes, Parent re-renders
  // ALL children re-render too!
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>
        Count: {count}
      </button>
      <Child />           {/* Re-renders */}
      <OtherChild />      {/* Re-renders */}
    </div>
  );
}
```

## Re-render vs DOM Update

```jsx
function Component() {
  const [count, setCount] = useState(0);
  
  // This re-renders every click
  // But only the span's textContent changes in DOM
  return (
    <div>
      <h1>Title</h1>       {/* DOM unchanged */}
      <span>{count}</span>  {/* DOM updated */}
      <p>Static text</p>   {/* DOM unchanged */}
    </div>
  );
}
```

React's diffing ensures minimal DOM updates even with full re-renders.

## Render Doesn't Mean Slow

Renders are usually fast:

```jsx
// This is fine for most cases
function List({ items }) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}
```

Optimize only when you have measured performance issues!

## Measuring Renders

```jsx
// Development: Count renders
function Component() {
  const renderCount = useRef(0);
  renderCount.current++;
  
  console.log(`Rendered ${renderCount.current} times`);
  
  return <div>...</div>;
}

// Or use React DevTools Profiler
// Components > Settings > "Highlight updates when components render"
```

---

> 📘 *This lesson is part of the [React Performance Deep Dive](https://stanza.dev/courses/react-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*