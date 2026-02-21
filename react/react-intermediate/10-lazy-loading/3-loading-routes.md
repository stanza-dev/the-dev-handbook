---
source_course: "react-intermediate"
source_lesson: "react-lazy-loading-routes"
---

# Lazy Loading Routes

## Introduction

Route-based lazy loading is the single most impactful code splitting technique for React applications. By loading each page only when the user navigates to it, you can dramatically reduce the initial JavaScript payload and improve Time to Interactive.

## Key Concepts

- **Dynamic route imports**: Using \`React.lazy\` to defer the import of page-level components until the route is matched.
- **Prefetching**: Loading a route's code before the user navigates, typically on link hover or during idle time.
- **Loading experience**: The fallback UI users see while a route chunk is downloading.

## Real World Context

An e-commerce site has a homepage, product listing, product detail, cart, and checkout. Each of these pages may import different third-party libraries (image galleries for products, payment forms for checkout). Without lazy routes, a first-time visitor downloads code for all five pages. With lazy routes, only the homepage code loads initially. When the user clicks a product, the product page chunk downloads on demand.

## Deep Dive

The pattern for lazy routes combines \`React.lazy\`, \`Suspense\`, and your routing library. Here is a comprehensive example showing route-level splitting with preloading:

\`\`\`tsx
import { lazy, Suspense } from 'react';
import { Routes, Route, Link } from 'react-router-dom';

const Home = lazy(() => import('./pages/Home'));
const Products = lazy(() => import('./pages/Products'));
const Checkout = lazy(() => import('./pages/Checkout'));

// Preloading function
const preloadCheckout = () => import('./pages/Checkout');

function NavBar() {
  return (
    <nav>
      <Link to="/">Home</Link>
      <Link to="/products">Products</Link>
      <Link
        to="/checkout"
        onMouseEnter={preloadCheckout}
      >
        Checkout
      </Link>
    </nav>
  );
}

function App() {
  return (
    <>
      <NavBar />
      <Suspense fallback={<PageSkeleton />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/products" element={<Products />} />
          <Route path="/checkout" element={<Checkout />} />
        </Routes>
      </Suspense>
    </>
  );
}
\`\`\`

Preloading is a powerful optimization. By calling \`import('./pages/Checkout')\` on hover, the browser starts downloading the chunk before the user clicks. If the download completes before navigation, the transition is instant. The import promise is cached by the bundler, so calling it multiple times does not trigger duplicate downloads.

For frameworks like Next.js, the \`next/link\` component automatically prefetches routes that are visible in the viewport, providing this optimization out of the box.

## Common Pitfalls

1. **Not providing a page-level Suspense** — Without a Suspense boundary around routes, a lazy route causes an error. Place Suspense at the router level.
2. **Showing a jarring loading state** — A simple spinner for the whole page is disorienting. Use skeleton screens that match the target page layout.

## Best Practices

1. **Preload on hover or viewport visibility** — Give the network a head start by triggering the import before the user commits to navigating.
2. **Use skeleton fallbacks** — Design fallbacks that resemble the page layout to reduce perceived loading time.

## Summary

- Lazy loading routes reduces the initial bundle by deferring page code until navigation.
- Preloading on hover or viewport visibility makes transitions feel instant.
- Use skeleton fallbacks for a smooth loading experience.

## Code Examples

**Route preloading on hover for instant transitions**

```tsx
import { lazy, Suspense } from 'react';
import { Link } from 'react-router-dom';

const Checkout = lazy(() => import('./pages/Checkout'));

const preloadCheckout = () => import('./pages/Checkout');

function CartButton() {
  return (
    <Link
      to="/checkout"
      onMouseEnter={preloadCheckout}
      onFocus={preloadCheckout}
    >
      Go to Checkout
    </Link>
  );
}
```


## Resources

- [lazy](https://react.dev/reference/react/lazy) — Official React.lazy API reference

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*