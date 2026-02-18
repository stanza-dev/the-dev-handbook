---
source_course: "react-data-fetching"
source_lesson: "react-data-fetching-why-data-fetching-is-different"
---

# Why Data Fetching is Different in React

Data fetching in React requires special consideration because React is declarative and component-based.

## The Challenge

Unlike traditional apps where you fetch data once on page load, React components can:
- Mount and unmount frequently
- Re-render many times
- Need data at different times
- Share data across components

## Key Concepts

### 1. Side Effects

Fetching data is a **side effect** - it's an action that happens outside React's rendering. React needs to know when to trigger these effects.

### 2. Loading States

Users need feedback while data loads:

```jsx
function UserProfile() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  // Three possible states:
  if (loading) return <Spinner />;
  if (error) return <Error message={error} />;
  return <Profile user={user} />;
}
```

### 3. Race Conditions

What if the user changes selection before data loads?

```jsx
// User clicks "Alice", then quickly clicks "Bob"
// Alice's data arrives after Bob's data
// Bug: Shows Alice's data when Bob is selected!
```

### 4. Memory Leaks

Setting state on unmounted components:

```jsx
// Component unmounts while fetching
// Then setState is called
// Warning: Can't perform state update on unmounted component
```

## The Data Fetching Lifecycle

```
1. Component Mounts
   ↓
2. Trigger Fetch (useEffect)
   ↓
3. Show Loading State
   ↓
4. Data Arrives (or Error)
   ↓
5. Update State
   ↓
6. Re-render with Data
```

## Modern Approaches

React offers several patterns for data fetching:

1. **useEffect** - Manual fetching in effects
2. **Suspense** - Declarative loading states
3. **Libraries** - TanStack Query, SWR, etc.
4. **Server Components** - Fetch on the server

We'll explore each approach in this course!

📚 **Learn more**: [Synchronizing with Effects](https://react.dev/learn/synchronizing-with-effects)

---

> 📘 *This lesson is part of the [React Data Fetching Patterns](https://stanza.dev/courses/react-data-fetching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*