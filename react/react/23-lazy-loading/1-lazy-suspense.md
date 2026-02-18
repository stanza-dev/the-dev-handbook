---
source_course: "react"
source_lesson: "react-lazy-suspense"
---

# Code Splitting with React.lazy

`React.lazy` lets you dynamically import components, reducing initial bundle size.

## How It Works

1. Wrap dynamic import with `React.lazy()`
2. Wrap lazy component with `<Suspense>`
3. Provide a fallback UI while loading

## When to Use

- Route-based splitting (different pages)
- Large components not needed immediately
- Modals and dialogs
- Admin sections

## Code Examples

**Lazy loading patterns**

```tsx
import { lazy, Suspense, useState } from 'react';

// Lazy load the component
const HeavyChart = lazy(() => import('./HeavyChart'));
const AdminPanel = lazy(() => import('./AdminPanel'));

// Loading component
function LoadingSpinner() {
  return <div className="spinner">Loading...</div>;
}

function App() {
  const [showChart, setShowChart] = useState(false);
  const [isAdmin, setIsAdmin] = useState(false);

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
      
      {isAdmin && (
        <Suspense fallback={<LoadingSpinner />}>
          <AdminPanel />
        </Suspense>
      )}
    </div>
  );
}

// Route-based code splitting
import { Routes, Route } from 'react-router-dom';

const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Dashboard = lazy(() => import('./pages/Dashboard'));

function AppRoutes() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/dashboard" element={<Dashboard />} />
      </Routes>
    </Suspense>
  );
}
```


## Resources

- [Code Splitting](https://react.dev/reference/react/lazy) — Official React.lazy documentation

---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*