---
source_course: "react-hooks-deep-dive"
source_lesson: "react-hooks-deep-dive-use-api"
---

# use(): Reading Resources in Render

React 19 introduces `use()`, a revolutionary API that lets you read resources (Promises and Context) during render. Unlike hooks, `use()` can be called conditionally.

## Reading Promises

```tsx
import { use, Suspense } from 'react';

async function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
}

function UserProfile({ userPromise }: { userPromise: Promise<User> }) {
  // use() suspends until the promise resolves
  const user = use(userPromise);

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}

function App() {
  const userPromise = fetchUser('123'); // Create promise in parent

  return (
    <Suspense fallback={<Loading />}>
      <UserProfile userPromise={userPromise} />
    </Suspense>
  );
}
```

## Reading Context Conditionally

Unlike `useContext`, `use()` can be called inside conditionals:

```tsx
function StatusMessage({ showDetails }: { showDetails: boolean }) {
  if (showDetails) {
    // This is allowed with use()!
    const theme = use(ThemeContext);
    return <div className={theme}>Detailed status...</div>;
  }
  return <div>Basic status</div>;
}
```

## Key Differences from Hooks

| Aspect | Hooks (useContext, etc.) | use() |
|--------|--------------------------|-------|
| Conditional calls | ❌ Not allowed | ✅ Allowed |
| Loops | ❌ Not allowed | ✅ Allowed |
| Promises | ❌ Not supported | ✅ Suspends |
| Early returns | ❌ Must be before | ✅ Can be after |

## Important Rules

1. **Create promises outside the component** or memoize them
2. **Wrap in Suspense** to handle the loading state
3. **Handle errors** with Error Boundaries

```tsx
// 🔴 Bad - creates new promise every render
function BadExample({ userId }) {
  const user = use(fetchUser(userId)); // Infinite loop!
}

// ✅ Good - promise created in parent
function GoodExample({ userPromise }) {
  const user = use(userPromise);
}

// ✅ Good - promise cached/memoized
function AlsoGood({ userId }) {
  const userPromise = useMemo(() => fetchUser(userId), [userId]);
  const user = use(userPromise);
}
```

## Resources

- [use API Reference](https://react.dev/reference/react/use) — Official React documentation for the use() API

---

> 📘 *This lesson is part of the [React Hooks Deep Dive](https://stanza.dev/courses/react-hooks-deep-dive) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*