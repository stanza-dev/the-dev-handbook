---
source_course: "react-performance"
source_lesson: "react-performance-react-profiler-api"
---

# Using the Profiler API

Programmatically measure render performance.

## The Profiler Component

```jsx
import { Profiler } from 'react';

function App() {
  const onRenderCallback = (
    id,                   // "id" prop of the Profiler
    phase,                // "mount" | "update" | "nested-update"
    actualDuration,       // Time spent rendering
    baseDuration,         // Estimated time without memoization
    startTime,            // When React started rendering
    commitTime,           // When React committed changes
  ) => {
    console.log({
      id,
      phase,
      actualDuration,
      baseDuration,
    });
  };

  return (
    <Profiler id="App" onRender={onRenderCallback}>
      <MainContent />
    </Profiler>
  );
}
```

## Nested Profilers

```jsx
function App() {
  return (
    <Profiler id="App" onRender={handleRender}>
      <Header />
      <Profiler id="MainContent" onRender={handleRender}>
        <ProductList />
      </Profiler>
      <Profiler id="Sidebar" onRender={handleRender}>
        <Sidebar />
      </Profiler>
    </Profiler>
  );
}
```

## Collecting Metrics

```jsx
const metrics = [];

function onRenderCallback(id, phase, actualDuration, baseDuration) {
  metrics.push({
    id,
    phase,
    actualDuration,
    baseDuration,
    timestamp: Date.now(),
  });
  
  // Warn on slow renders
  if (actualDuration > 16) { // More than 1 frame at 60fps
    console.warn(`Slow render: ${id} took ${actualDuration.toFixed(2)}ms`);
  }
}

// Send to analytics
window.addEventListener('beforeunload', () => {
  if (metrics.length > 0) {
    navigator.sendBeacon('/api/metrics', JSON.stringify(metrics));
  }
});
```

## Production Profiling

```jsx
// Create a profiling build
// In package.json scripts:
{
  "build:profile": "react-scripts build --profile"
}
```

Or with Vite:
```js
// vite.config.js
export default defineConfig({
  build: {
    sourcemap: true,
  },
  define: {
    'process.env.ENABLE_PROFILING': 'true',
  },
});
```

## Custom Performance Marks

```jsx
function ExpensiveOperation() {
  useEffect(() => {
    performance.mark('expensive-start');
    
    // Do expensive work
    doExpensiveWork();
    
    performance.mark('expensive-end');
    performance.measure(
      'expensive-operation',
      'expensive-start',
      'expensive-end'
    );
    
    const measure = performance.getEntriesByName('expensive-operation')[0];
    console.log(`Operation took ${measure.duration}ms`);
  }, []);
}
```

## React DevTools Profiler Settings

1. **Record why each component rendered**
2. **Hide commits below threshold**
3. **Show native elements**

Access via: Components → Settings (gear icon)

---

> 📘 *This lesson is part of the [React Performance Deep Dive](https://stanza.dev/courses/react-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*