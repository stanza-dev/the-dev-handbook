---
source_course: "react"
source_lesson: "react-lazy-code-splitting"
---

# Code Splitting with React.lazy

## Introduction

Code splitting breaks your application into smaller chunks that load on demand. `React.lazy` makes this easy by letting you dynamically import components.

## Key Concepts

**React.lazy** accepts a function that returns a dynamic import and returns a component that can be rendered inside a Suspense boundary.

```tsx
const MyComponent = lazy(() => import('./MyComponent'));
```

## Real World Context

Code splitting is essential for:
- Reducing initial bundle size for faster page loads
- Loading features only when users need them
- Route-based splitting in single-page applications
- Conditional feature loading based on user permissions

## Deep Dive

### How It Works

1. `lazy()` creates a component that triggers a dynamic import on first render
2. The import returns a promise for a module with a default export
3. While loading, the component suspends
4. Once loaded, the component renders normally
5. The module is cached - subsequent renders don't re-fetch

### Route-Based Code Splitting

The most impactful pattern - split by route:

```tsx
import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';

const Home = lazy(() => import('./pages/Home'));
const Products = lazy(() => import('./pages/Products'));
const Checkout = lazy(() => import('./pages/Checkout'));
const Admin = lazy(() => import('./pages/Admin'));

function App() {
  return (
    <Suspense fallback={<PageLoader />}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/products" element={<Products />} />
        <Route path="/checkout" element={<Checkout />} />
        <Route path="/admin" element={<Admin />} />
      </Routes>
    </Suspense>
  );
}
```

### Named Exports

`lazy` expects a default export. For named exports:

```tsx
// If the module exports { MyChart }
const Chart = lazy(() =>
  import('./Charts').then(module => ({ default: module.MyChart }))
);
```

### Preloading Components

Preload before the user needs them:

```tsx
const Checkout = lazy(() => import('./Checkout'));

// Preload when user hovers "Buy" button
function ProductCard() {
  const preloadCheckout = () => {
    import('./Checkout');
  };

  return (
    <button
      onMouseEnter={preloadCheckout}
      onClick={goToCheckout}
    >
      Buy Now
    </button>
  );
}
```

## Common Pitfalls

1. **Over-splitting**: Every lazy component adds a network request. Group related components together.
2. **Forgetting Suspense**: lazy components MUST be inside a Suspense boundary or React throws an error.
3. **Server-side rendering issues**: React.lazy doesn't work in SSR out of the box. Use framework solutions (Next.js dynamic) or libraries.

## Best Practices

- Split at route boundaries first - biggest impact with least complexity
- Split large feature modules (admin dashboards, editors, charts)
- Don't split small components - the overhead isn't worth it
- Preload components the user is likely to need next
- Use loading skeletons that match the component's layout
- Consider using error boundaries to handle failed imports

## Summary

`React.lazy` enables code splitting by dynamically importing components. Combined with Suspense, it provides a seamless experience for loading parts of your app on demand. Focus on route-level splitting for the biggest performance gains.

## Code Examples

**Code splitting with lazy loading and preloading**

```tsx
import { lazy, Suspense, useState } from 'react';

// Split heavy components
const RichTextEditor = lazy(() => import('./RichTextEditor'));
const DataVisualization = lazy(() => import('./DataVisualization'));
const AdminPanel = lazy(() => import('./AdminPanel'));

// Loading component
function ComponentLoader() {
  return (
    <div className="loader">
      <div className="spinner" />
      <p>Loading component...</p>
    </div>
  );
}

function FeaturePanel({ feature }: { feature: string }) {
  return (
    <Suspense fallback={<ComponentLoader />}>
      {feature === 'editor' && <RichTextEditor />}
      {feature === 'charts' && <DataVisualization />}
      {feature === 'admin' && <AdminPanel />}
    </Suspense>
  );
}

// Preloading pattern
const preloadEditor = () => import('./RichTextEditor');

function Toolbar() {
  return (
    <nav>
      <button
        onMouseEnter={preloadEditor}
        onClick={() => showFeature('editor')}
      >
        Open Editor
      </button>
    </nav>
  );
}
```


## Resources

- [lazy Reference](https://react.dev/reference/react/lazy) — Official React.lazy documentation
- [Code Splitting](https://react.dev/reference/react/lazy#suspense-for-code-splitting) — Guide to code splitting with React

---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*