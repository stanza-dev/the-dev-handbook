---
source_course: "react-advanced-apis"
source_lesson: "react-advanced-apis-preload-apis"
---

# Resource Preloading: Optimizing Load Performance

React 19 provides APIs to preload resources like scripts, stylesheets, and fonts before they're needed.

## prefetchDNS

Start DNS lookup early for domains you'll connect to:

```tsx
import { prefetchDNS } from 'react-dom';

function App() {
  prefetchDNS('https://api.example.com');
  prefetchDNS('https://cdn.example.com');
  
  return <MainContent />;
}
```

## preconnect

Establish early connections (DNS + TCP + TLS):

```tsx
import { preconnect } from 'react-dom';

function App() {
  // Full connection setup before needed
  preconnect('https://api.example.com');
  
  return <MainContent />;
}
```

## preload

Start downloading a resource you'll need soon:

```tsx
import { preload } from 'react-dom';

function App() {
  // Preload a stylesheet
  preload('/styles/dashboard.css', { as: 'style' });
  
  // Preload a font
  preload('/fonts/inter.woff2', { as: 'font', type: 'font/woff2' });
  
  // Preload an image
  preload('/hero.jpg', { as: 'image' });
  
  return <MainContent />;
}
```

## preinit

Fetch AND execute a script or stylesheet immediately:

```tsx
import { preinit } from 'react-dom';

function App() {
  // Download and execute immediately
  preinit('/analytics.js', { as: 'script' });
  preinit('/critical.css', { as: 'style' });
  
  return <MainContent />;
}
```

## preloadModule / preinitModule

For ES modules specifically:

```tsx
import { preloadModule, preinitModule } from 'react-dom';

function App() {
  // Preload an ES module
  preloadModule('/modules/charts.js');
  
  // Preload and execute an ES module
  preinitModule('/modules/analytics.js');
  
  return <MainContent />;
}
```

## Practical Example: Route Preloading

```tsx
import { preload, preloadModule } from 'react-dom';

const Dashboard = lazy(() => import('./Dashboard'));

function NavLink({ to, children }) {
  const handleMouseEnter = () => {
    if (to === '/dashboard') {
      // Preload the route's resources
      preloadModule('./Dashboard');
      preload('/api/dashboard-data', { as: 'fetch' });
      preload('/styles/dashboard.css', { as: 'style' });
    }
  };
  
  return (
    <Link to={to} onMouseEnter={handleMouseEnter}>
      {children}
    </Link>
  );
}
```

## Comparison Table

| API | DNS | TCP | TLS | Download | Execute |
|-----|-----|-----|-----|----------|--------|
| prefetchDNS | ✅ | ❌ | ❌ | ❌ | ❌ |
| preconnect | ✅ | ✅ | ✅ | ❌ | ❌ |
| preload | ✅ | ✅ | ✅ | ✅ | ❌ |
| preinit | ✅ | ✅ | ✅ | ✅ | ✅ |

## Resources

- [preload API Reference](https://react.dev/reference/react-dom/preload) — Official React documentation for preload
- [preinit API Reference](https://react.dev/reference/react-dom/preinit) — Official React documentation for preinit

---

> 📘 *This lesson is part of the [React Advanced APIs](https://stanza.dev/courses/react-advanced-apis) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*