---
source_course: "react-advanced-apis"
source_lesson: "react-advanced-apis-react-lazy"
---

# lazy: Code Splitting Components

`lazy` lets you defer loading a component's code until it's rendered for the first time. This reduces your initial bundle size.

## Basic Usage

```tsx
import { lazy, Suspense } from 'react';

// Instead of: import MarkdownPreview from './MarkdownPreview';
const MarkdownPreview = lazy(() => import('./MarkdownPreview'));

function Editor() {
  const [showPreview, setShowPreview] = useState(false);
  
  return (
    <>
      <button onClick={() => setShowPreview(true)}>
        Show Preview
      </button>
      {showPreview && (
        <Suspense fallback={<Loading />}>
          <MarkdownPreview />
        </Suspense>
      )}
    </>
  );
}
```

## How It Works

1. **Initial load**: `MarkdownPreview` code is NOT in the bundle
2. **User clicks button**: React starts loading the component
3. **While loading**: Suspense shows the fallback
4. **Load complete**: Component renders normally

## Requirements

### Default Exports

`lazy` expects the module to have a default export:

```tsx
// MarkdownPreview.tsx
export default function MarkdownPreview() {
  return <div>...</div>;
}

// ✅ Works
const MarkdownPreview = lazy(() => import('./MarkdownPreview'));
```

### Named Exports

For named exports, re-export as default:

```tsx
// utils.tsx
export function MarkdownPreview() { ... }

// To use with lazy:
const MarkdownPreview = lazy(() =>
  import('./utils').then(module => ({ default: module.MarkdownPreview }))
);
```

## Route-Based Code Splitting

The most common pattern - split by route:

```tsx
import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';

const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Dashboard = lazy(() => import('./pages/Dashboard'));

function App() {
  return (
    <Suspense fallback={<PageLoader />}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/dashboard" element={<Dashboard />} />
      </Routes>
    </Suspense>
  );
}
```

## Preloading Components

Load components before they're needed:

```tsx
const Dashboard = lazy(() => import('./Dashboard'));

// Preload on hover
function NavLink() {
  const preload = () => {
    import('./Dashboard'); // Start loading
  };
  
  return (
    <Link to="/dashboard" onMouseEnter={preload}>
      Dashboard
    </Link>
  );
}
```

## Error Handling

Wrap lazy components in Error Boundaries:

```tsx
<ErrorBoundary fallback={<LoadError />}>
  <Suspense fallback={<Loading />}>
    <LazyComponent />
  </Suspense>
</ErrorBoundary>
```

## Resources

- [lazy API Reference](https://react.dev/reference/react/lazy) — Official React documentation for lazy

---

> 📘 *This lesson is part of the [React Advanced APIs](https://stanza.dev/courses/react-advanced-apis) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*