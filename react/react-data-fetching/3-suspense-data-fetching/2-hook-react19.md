---
source_course: "react-data-fetching"
source_lesson: "react-data-fetching-use-hook-react19"
---

# The use() Hook in React 19

React 19 introduces `use()` - a special hook that can read resources like promises and context.

## Basic Syntax

```jsx
import { use } from 'react';

function UserProfile({ userPromise }) {
  const user = use(userPromise);
  return <h1>{user.name}</h1>;
}
```

## With Data Fetching

```jsx
import { use, Suspense } from 'react';

// Cache for promises
const cache = new Map();

function fetchData(url) {
  if (!cache.has(url)) {
    cache.set(url, fetch(url).then(r => r.json()));
  }
  return cache.get(url);
}

function UserProfile({ userId }) {
  const user = use(fetchData(`/api/users/${userId}`));
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <UserProfile userId="123" />
    </Suspense>
  );
}
```

## use() vs useEffect

| Feature | use() | useEffect |
|---------|-------|----------|
| Suspense integration | ✅ | ❌ |
| Conditional calls | ✅ | ❌ |
| In loops | ✅ | ❌ |
| Loading state | Automatic | Manual |
| Error boundaries | Automatic | Manual |

## Conditional use()

Unlike other hooks, `use()` can be called conditionally:

```jsx
function OptionalData({ shouldFetch }) {
  if (shouldFetch) {
    const data = use(fetchData('/api/data'));
    return <DataView data={data} />;
  }
  return <p>Click to load data</p>;
}
```

## use() with Context

`use()` can also read context values:

```jsx
import { use, createContext } from 'react';

const ThemeContext = createContext('light');

function Button() {
  // Can be called conditionally!
  const theme = use(ThemeContext);
  return <button className={theme}>Click me</button>;
}
```

## Error Handling with Error Boundaries

```jsx
import { ErrorBoundary } from 'react-error-boundary';

function App() {
  return (
    <ErrorBoundary fallback={<ErrorMessage />}>
      <Suspense fallback={<Loading />}>
        <DataComponent />
      </Suspense>
    </ErrorBoundary>
  );
}
```

If the promise rejects, the error boundary catches it automatically.

---

> 📘 *This lesson is part of the [React Data Fetching Patterns](https://stanza.dev/courses/react-data-fetching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*