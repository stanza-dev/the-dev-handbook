---
source_course: "react-performance"
source_lesson: "react-performance-lazy-loading"
---

# React.lazy and Suspense

Load components on demand to reduce initial bundle size.

## Basic Lazy Loading

```jsx
import { lazy, Suspense } from 'react';

// Instead of: import HeavyComponent from './HeavyComponent';
const HeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <HeavyComponent />
    </Suspense>
  );
}
```

## Route-Based Splitting

```jsx
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

// Each route in its own chunk
const Home = lazy(() => import('./pages/Home'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));
const AdminPanel = lazy(() => import('./pages/AdminPanel'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<PageLoader />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/settings" element={<Settings />} />
          <Route path="/admin" element={<AdminPanel />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

## Named Exports with lazy

```jsx
// Default export works directly
const Modal = lazy(() => import('./Modal'));

// Named exports need intermediate step
const Chart = lazy(() =>
  import('./Charts').then(module => ({ default: module.BarChart }))
);

// Or create a re-export file
// ChartExports.js
export { BarChart as default } from './Charts';

// Then
const BarChart = lazy(() => import('./ChartExports'));
```

## Prefetching

```jsx
// Prefetch on hover
function NavLink({ to, children }) {
  const prefetch = () => {
    // Trigger the lazy import
    if (to === '/dashboard') {
      import('./pages/Dashboard');
    }
  };
  
  return (
    <Link to={to} onMouseEnter={prefetch}>
      {children}
    </Link>
  );
}
```

## Conditional Lazy Loading

```jsx
function FeatureSection({ showAdvanced }) {
  // Only load when actually needed
  const AdvancedFeatures = lazy(() => import('./AdvancedFeatures'));
  
  return (
    <div>
      <BasicFeatures />
      {showAdvanced && (
        <Suspense fallback={<Loading />}>
          <AdvancedFeatures />
        </Suspense>
      )}
    </div>
  );
}
```

## Error Boundaries with Suspense

```jsx
import { lazy, Suspense } from 'react';
import { ErrorBoundary } from 'react-error-boundary';

const HeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <ErrorBoundary fallback={<Error />}>
      <Suspense fallback={<Loading />}>
        <HeavyComponent />
      </Suspense>
    </ErrorBoundary>
  );
}
```

## Multiple Suspense Boundaries

```jsx
function Dashboard() {
  return (
    <div>
      <Header /> {/* Always loaded */}
      
      <Suspense fallback={<ChartSkeleton />}>
        <Charts />
      </Suspense>
      
      <Suspense fallback={<TableSkeleton />}>
        <DataTable />
      </Suspense>
      
      <Footer /> {/* Always loaded */}
    </div>
  );
}
```

---

> 📘 *This lesson is part of the [React Performance Deep Dive](https://stanza.dev/courses/react-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*