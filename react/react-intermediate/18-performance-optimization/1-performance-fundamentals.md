---
source_course: "react-intermediate"
source_lesson: "react-performance-fundamentals"
---

# Understanding React's Rendering Behavior

## Introduction

Before optimizing, you need to understand what triggers renders and why. Most performance issues come from unnecessary renders, not slow renders.

## Key Concepts

**Rendering** in React means calling your component function. A **re-render** happens when React calls your component again to get the latest JSX. Rendering doesn't necessarily mean DOM updates.

## Real World Context

In production apps:
- Understanding renders helps you find performance bottlenecks
- Most "slow" apps render too often, not too slowly
- Premature optimization causes more problems than it solves
- Profiling before optimizing saves wasted effort

## Deep Dive

### What Triggers a Re-render?

1. **State changes**: `setState` causes the component to re-render
2. **Parent re-renders**: When a parent re-renders, all children re-render
3. **Context changes**: When a context value changes, all consumers re-render
4. **Hook changes**: Some hooks (like useReducer dispatch) trigger re-renders

### The Render → Commit Cycle

```
Trigger → Render → Reconciliation → Commit
   ↓        ↓           ↓             ↓
 setState  Call      Compare      Update
           functions  old/new     actual
                      JSX         DOM
```

1. **Trigger**: Something requests an update (setState, context change)
2. **Render**: React calls component functions to get new JSX
3. **Reconciliation**: React compares old and new virtual DOM
4. **Commit**: Only changed parts are updated in the real DOM

### What Doesn't Cause Re-renders

- Ref changes (`useRef`)
- Reading from refs
- Mutating objects (though this causes bugs!)
- Props passed to memoized components (if unchanged)

### Measuring Render Performance

```tsx
import { Profiler } from 'react';

function onRender(
  id: string,
  phase: 'mount' | 'update',
  actualDuration: number,
  baseDuration: number
) {
  console.log(`${id} ${phase}: ${actualDuration.toFixed(2)}ms`);
}

<Profiler id="App" onRender={onRender}>
  <App />
</Profiler>
```

### React DevTools Profiler

The Profiler tab in React DevTools shows:
- Which components rendered
- How long each render took
- Why components re-rendered
- Render frequency over time

## Common Pitfalls

1. **Optimizing before measuring**: Always profile first. Many "optimizations" don't improve real performance.
2. **Thinking renders are expensive**: React's reconciliation is fast. Only optimize actual bottlenecks.
3. **Blaming React for slow code**: Often the issue is expensive computations inside components, not React itself.

## Best Practices

- Profile before optimizing - use React DevTools Profiler
- Focus on actual user-perceived slowness, not render counts
- Understand why components re-render before preventing it
- Keep component trees flat when possible
- Measure the impact of optimizations
- Don't memo everything - it has costs too

## Summary

React re-renders when state changes, parents re-render, or context changes. Use the DevTools Profiler to measure actual performance before optimizing. Most performance issues are unnecessary renders, not slow renders.

## Code Examples

**Profiling and understanding renders**

```tsx
import { Profiler, useState, useEffect } from 'react';

// Track renders in development
function RenderCounter({ name }: { name: string }) {
  const renderCount = useRef(0);
  renderCount.current++;
  
  useEffect(() => {
    console.log(`${name} rendered ${renderCount.current} times`);
  });
  
  return null;
}

// Use Profiler to measure performance
function ProfiledApp() {
  function handleRender(
    id: string,
    phase: 'mount' | 'update',
    actualDuration: number
  ) {
    // Only log slow renders
    if (actualDuration > 16) { // Longer than a frame
      console.warn(`Slow render: ${id} took ${actualDuration.toFixed(2)}ms`);
    }
  }

  return (
    <Profiler id="App" onRender={handleRender}>
      <App />
    </Profiler>
  );
}

// Common re-render causes
function ParentChild() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>
        Count: {count}
      </button>
      
      {/* This re-renders on every parent render */}
      {/* Even though it doesn't use count */}
      <ExpensiveChild />
      
      {/* To prevent, use memo */}
      <MemoizedExpensiveChild />
    </div>
  );
}
```


## Resources

- [React Profiler](https://react.dev/reference/react/Profiler) — Official Profiler component documentation
- [React DevTools Profiler](https://react.dev/learn/react-developer-tools#profiling-components-with-the-profiler) — Using the React DevTools Profiler

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*