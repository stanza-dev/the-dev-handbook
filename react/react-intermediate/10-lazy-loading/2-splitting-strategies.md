---
source_course: "react-intermediate"
source_lesson: "react-code-splitting-strategies"
---

# Code Splitting Strategies

## Introduction

Not all code splitting is created equal. The strategy you choose determines how much impact lazy loading has on your application's performance. This lesson explores the most effective strategies and how to decide what to split.

## Key Concepts

- **Route-based splitting**: Loading different pages as separate chunks, the most common and impactful strategy.
- **Component-based splitting**: Deferring heavy components like editors, charts, or maps that are not immediately visible.
- **Bundle analysis**: Using tools to visualize your bundle and identify the largest modules worth splitting.

## Real World Context

A SaaS dashboard has a landing page, a settings page, a data visualization page with heavy chart libraries, and an admin page. Route-based splitting means each page is its own chunk. The chart library, which is 400KB, only loads when the user visits the data page. The admin page code is never fetched for regular users.

## Deep Dive

**Route-based splitting** is the most impactful strategy because routes are natural boundaries where users expect a brief loading pause. With React Router or Next.js, each route can be a lazy component:

\`\`\`tsx
import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';

const Home = lazy(() => import('./pages/Home'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <Suspense fallback={<PageLoader />}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}
\`\`\`

**Component-based splitting** targets expensive components that appear conditionally. Modals, rich text editors, chart libraries, and admin panels are good candidates because they are either large or infrequently accessed.

**Bundle analysis** helps identify what to split. Tools like \`webpack-bundle-analyzer\` or \`@next/bundle-analyzer\` generate treemaps showing the size of every module in your bundle. Look for the largest third-party libraries and feature modules.

## Common Pitfalls

1. **Splitting too aggressively** — Every lazy import creates a separate network request. Splitting dozens of tiny components adds latency from request overhead.
2. **Not analyzing the bundle first** — Without data, you might split components that are already small while missing the real bottlenecks.

## Best Practices

1. **Use bundle analysis tools** — Visualize your bundle before deciding what to split. Target the largest modules first.
2. **Group related components in the same chunk** — Use dynamic import comments like \`/* webpackChunkName: "admin" */\` to group related lazy components into a single chunk.

## Summary

- Route-based splitting provides the biggest impact with natural loading boundaries.
- Component-based splitting targets heavy, conditionally-rendered features.
- Always analyze your bundle to make data-driven splitting decisions.

## Code Examples

**Route-based code splitting with React Router**

```tsx
import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';

const Home = lazy(() => import('./pages/Home'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <Suspense fallback={<PageLoader />}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}
```


## Resources

- [Suspense for code splitting](https://react.dev/reference/react/lazy#suspense-for-code-splitting) — Guide to code splitting with React.lazy and Suspense

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*