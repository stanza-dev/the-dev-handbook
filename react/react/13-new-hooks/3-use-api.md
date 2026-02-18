---
source_course: "react"
source_lesson: "react-use-api"
---

# The `use` API

Read values from Promises or Context during render. Unlike other hooks, `use` can be called conditionally!

## Reading Promises

```jsx
const data = use(dataPromise);
```

- Suspends until promise resolves
- Must be used with Suspense boundary
- Promise should be cached (not created in render)

## Reading Context Conditionally

```jsx
function Heading({ showTitle }) {
  if (!showTitle) return null;
  
  // This works! Can't do this with useContext
  const theme = use(ThemeContext);
  return <h1 style={{ color: theme.primary }}>Hello</h1>;
}
```

## Key Difference from Hooks

- Can be called inside `if` statements
- Can be called inside loops
- Follows different rules than hooks

## Code Examples

**use API for Promises and conditional Context**

```tsx
import { use, Suspense } from 'react';

// Reading a Promise
function UserProfile({ userPromise }) {
  // Suspends until resolved
  const user = use(userPromise);
  
  return (
    <div className="profile">
      <img src={user.avatar} alt={user.name} />
      <h2>{user.name}</h2>
      <p>{user.bio}</p>
    </div>
  );
}

function App({ userId }) {
  // Create promise outside render or cache it
  const userPromise = fetchUser(userId);
  
  return (
    <Suspense fallback={<ProfileSkeleton />}>
      <UserProfile userPromise={userPromise} />
    </Suspense>
  );
}

// Conditional Context reading (not possible with useContext)
import { ThemeContext } from './contexts';

function ConditionalThemedBox({ useTheme, children }) {
  // Early return before hook - this is allowed with use()!
  if (!useTheme) {
    return <div className="default-box">{children}</div>;
  }
  
  const theme = use(ThemeContext);
  
  return (
    <div style={{ 
      background: theme.background,
      color: theme.text 
    }}>
      {children}
    </div>
  );
}
```


---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*