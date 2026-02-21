---
source_course: "react-intermediate"
source_lesson: "react-lazy-suspense"
---

# React.lazy and Suspense

## Introduction

As React applications grow, the JavaScript bundle can become large enough to slow down the initial page load significantly. React.lazy and Suspense work together to solve this by loading components on demand, splitting your code into smaller chunks that are fetched only when needed.

## Key Concepts

- **React.lazy**: A function that takes a dynamic import and returns a lazy-loaded component. The import is deferred until the component is first rendered.
- **Suspense**: A component that wraps lazy-loaded children and shows a fallback UI while they are being fetched.
- **Code splitting**: Breaking your application bundle into smaller chunks that are loaded independently.

## Real World Context

Consider a web application with an admin panel that most users never access. Without code splitting, every user downloads the admin code. With React.lazy, the admin panel code is only fetched when a user navigates to that route, reducing the initial bundle by potentially hundreds of kilobytes.

## Deep Dive

React.lazy accepts a function that must return a Promise resolving to a module with a default export containing a React component. Suspense catches the promise thrown during the lazy component's first render and displays the fallback until it resolves.

\`\`\`tsx
import { lazy, Suspense, useState } from 'react';

const HeavyChart = lazy(() => import('./HeavyChart'));
const AdminPanel = lazy(() => import('./AdminPanel'));

function LoadingSpinner() {
  return <div className="spinner">Loading...</div>;
}

function App() {
  const [showChart, setShowChart] = useState(false);

  return (
    <div>
      <button onClick={() => setShowChart(true)}>
        Show Chart
      </button>

      {showChart && (
        <Suspense fallback={<LoadingSpinner />}>
          <HeavyChart data={chartData} />
        </Suspense>
      )}
    </div>
  );
}
\`\`\`

React.lazy only supports default exports. For named exports, you can re-export them as defaults or use a wrapper:

\`\`\`tsx
const MyChart = lazy(() =>
  import('./Charts').then(mod => ({ default: mod.MyChart }))
);
\`\`\`

You can also preload components before the user needs them by calling the dynamic import early, such as on mouse hover or route prefetch.

## Common Pitfalls

1. **Forgetting the Suspense wrapper** — A lazy component rendered without a Suspense ancestor causes a runtime error. Always wrap lazy components in Suspense.
2. **Over-splitting small components** — Each lazy component adds a network request. Only split large features or routes, not tiny UI elements.

## Best Practices

1. **Split at route boundaries first** — Route-based splitting provides the biggest improvement with the least complexity.
2. **Preload on hover or focus** — Call \`import('./Component')\` on mouse enter events so the chunk downloads before the user clicks.

## Summary

- React.lazy defers component loading until first render, reducing initial bundle size.
- Suspense provides a fallback UI while lazy components are being fetched.
- Focus code splitting on routes and large features for maximum impact.

## Code Examples

**Lazy loading a heavy component with Suspense fallback**

```tsx
import { lazy, Suspense, useState } from 'react';

const HeavyChart = lazy(() => import('./HeavyChart'));

function App() {
  const [showChart, setShowChart] = useState(false);

  return (
    <div>
      <button onClick={() => setShowChart(true)}>Show Chart</button>
      {showChart && (
        <Suspense fallback={<div>Loading chart...</div>}>
          <HeavyChart />
        </Suspense>
      )}
    </div>
  );
}
```


## Resources

- [lazy](https://react.dev/reference/react/lazy) — Official React.lazy documentation

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*