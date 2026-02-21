---
source_course: "react-intermediate"
source_lesson: "react-suspense-fundamentals"
---

# Suspense Fundamentals

## Introduction

Suspense is React's mechanism for declaratively handling asynchronous operations. Instead of managing loading states manually, you declare what to show while waiting.

## Key Concepts

**Suspense** is a component that "catches" promises thrown during render and displays a fallback until they resolve. It transforms imperative loading logic into declarative UI boundaries.

```tsx
<Suspense fallback={<Loading />}>
  <AsyncComponent />
</Suspense>
```

## Real World Context

In production applications, Suspense enables:
- Skeleton loading screens that match your final UI
- Streaming server-side rendering for faster initial loads
- Code splitting with lazy loading
- Coordinated loading states across multiple components

## Deep Dive

### How Suspense Works

1. A component "suspends" by throwing a promise during render
2. React catches the promise at the nearest Suspense boundary
3. The fallback is shown until the promise resolves
4. React retries rendering the suspended component
5. The actual content is displayed

### What Can Suspend?

- **React.lazy()** - Code-split components
- **use()** - Reading promises (React 19)
- **Data fetching libraries** - Those that integrate with Suspense (TanStack Query, Relay, etc.)
- **Server Components** - Async components in RSC environments

### Fallback Prop

The `fallback` prop accepts any React node:

```tsx
// Simple spinner
<Suspense fallback={<Spinner />}>

// Skeleton matching final UI
<Suspense fallback={<ProfileSkeleton />}>

// Null for invisible loading (use carefully)
<Suspense fallback={null}>
```

### Nesting Suspense Boundaries

Nested boundaries create a waterfall of loading states:

```tsx
<Suspense fallback={<PageSkeleton />}>
  <Header />
  <Suspense fallback={<ContentSkeleton />}>
    <Content />
    <Suspense fallback={<CommentsSkeleton />}>
      <Comments />
    </Suspense>
  </Suspense>
</Suspense>
```

## Common Pitfalls

1. **Suspense waterfall**: Nesting too many boundaries causes sequential loading instead of parallel. Group related content under the same boundary.
2. **Missing fallback**: Suspense requires a fallback prop. Without it, React will look for a parent Suspense or throw an error.
3. **Not all async is Suspense-compatible**: Regular promises and useEffect fetching don't trigger Suspense - you need specific integrations.

## Best Practices

- Match fallback skeletons to your actual UI layout for smooth transitions
- Group related content under single boundaries to avoid loading waterfalls
- Place boundaries at meaningful UI divisions (page sections, cards, modals)
- Use libraries that support Suspense for data fetching (TanStack Query, SWR)
- Keep fallbacks lightweight to minimize time-to-interactive

## Summary

Suspense provides declarative loading states by catching promises thrown during render. Wrap async content in Suspense boundaries with appropriate fallbacks to create smooth loading experiences.

## Code Examples

**Suspense with lazy-loaded components and skeleton fallbacks**

```tsx
import { Suspense, lazy } from 'react';

// Lazy-loaded component
const Dashboard = lazy(() => import('./Dashboard'));
const Analytics = lazy(() => import('./Analytics'));

// Skeleton components
function DashboardSkeleton() {
  return (
    <div className="dashboard-skeleton">
      <div className="skeleton-header" />
      <div className="skeleton-cards">
        {[1, 2, 3].map(i => (
          <div key={i} className="skeleton-card" />
        ))}
      </div>
    </div>
  );
}

function AnalyticsSkeleton() {
  return <div className="skeleton-chart" />;
}

function App() {
  return (
    <div className="app">
      <Suspense fallback={<DashboardSkeleton />}>
        <Dashboard />
        
        {/* Nested Suspense for independent loading */}
        <Suspense fallback={<AnalyticsSkeleton />}>
          <Analytics />
        </Suspense>
      </Suspense>
    </div>
  );
}
```


## Resources

- [Suspense Reference](https://react.dev/reference/react/Suspense) — Official React Suspense documentation

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*