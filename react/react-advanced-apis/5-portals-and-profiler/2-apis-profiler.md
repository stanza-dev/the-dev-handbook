---
source_course: "react-advanced-apis"
source_lesson: "react-advanced-apis-profiler"
---

# Profiler: Measuring Render Performance

`<Profiler>` lets you measure rendering performance of a React tree programmatically.

## Basic Usage

```tsx
import { Profiler } from 'react';

function onRender(
  id,           // The "id" prop of the Profiler tree
  phase,        // "mount" | "update" | "nested-update"
  actualDuration,   // Time spent rendering the committed update
  baseDuration,     // Estimated time to render without memoization
  startTime,        // When React began rendering this update
  commitTime        // When React committed this update
) {
  console.log(`${id} ${phase}: ${actualDuration.toFixed(2)}ms`);
}

function App() {
  return (
    <Profiler id="App" onRender={onRender}>
      <Header />
      <Main />
      <Footer />
    </Profiler>
  );
}
```

## Understanding the Callback Parameters

| Parameter | Description |
|-----------|-------------|
| `id` | Identifies which Profiler tree was measured |
| `phase` | `"mount"` (first render), `"update"`, or `"nested-update"` |
| `actualDuration` | Time spent rendering (what you want to minimize) |
| `baseDuration` | Worst-case render time without memoization |
| `startTime` | Timestamp when React started rendering |
| `commitTime` | Timestamp when React committed the update |

## Nested Profilers

Measure different parts of your app:

```tsx
function App() {
  return (
    <Profiler id="App" onRender={onRender}>
      <Profiler id="Sidebar" onRender={onRender}>
        <Sidebar />
      </Profiler>
      <Profiler id="MainContent" onRender={onRender}>
        <MainContent />
      </Profiler>
    </Profiler>
  );
}
```

## Production Profiling

By default, Profiler is disabled in production. To enable:

```bash
# React build with profiling
npm run build -- --profile
```

Or use the profiling build:

```tsx
// webpack.config.js
resolve: {
  alias: {
    'react-dom$': 'react-dom/profiling',
    'scheduler/tracing': 'scheduler/tracing-profiling'
  }
}
```

## Practical Example: Performance Monitoring

```tsx
function sendToAnalytics(metrics) {
  // Send to your analytics service
  fetch('/api/performance', {
    method: 'POST',
    body: JSON.stringify(metrics)
  });
}

function onRender(id, phase, actualDuration, baseDuration) {
  // Only report slow renders
  if (actualDuration > 16) { // Longer than one frame
    sendToAnalytics({
      component: id,
      phase,
      duration: actualDuration,
      timestamp: Date.now()
    });
  }
}

function App() {
  return (
    <Profiler id="App" onRender={onRender}>
      <SlowComponent />
    </Profiler>
  );
}
```

## When to Use Profiler

✅ **Use for:**
- Identifying slow components
- Measuring optimization impact
- Production performance monitoring
- A/B testing render performance

❌ **Don't use for:**
- General debugging (use React DevTools)
- Measuring non-render performance
- Every component (adds overhead)

## Resources

- [Profiler API Reference](https://react.dev/reference/react/Profiler) — Official React documentation for Profiler

---

> 📘 *This lesson is part of the [React Advanced APIs](https://stanza.dev/courses/react-advanced-apis) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*