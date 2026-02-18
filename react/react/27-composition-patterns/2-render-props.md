---
source_course: "react"
source_lesson: "react-render-props"
---

# Render Props Pattern

A render prop is a function prop that a component uses to know what to render.

## When to Use

- When you need to share behavior, not just UI
- For headless components (logic only)
- When hooks aren't suitable

## Modern Alternative

Custom hooks have largely replaced render props, but the pattern is still useful for some use cases.

## Code Examples

**Render props patterns**

```tsx
import { useState, useEffect, ReactNode } from 'react';

// Render prop component for mouse position
type MousePosition = { x: number; y: number };

function MouseTracker({ 
  render 
}: { 
  render: (position: MousePosition) => ReactNode 
}) {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handleMove = (e: MouseEvent) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };
    
    window.addEventListener('mousemove', handleMove);
    return () => window.removeEventListener('mousemove', handleMove);
  }, []);

  return <>{render(position)}</>;
}

// Usage - you control the rendering
function App() {
  return (
    <MouseTracker
      render={({ x, y }) => (
        <div>
          <h1>Mouse Position</h1>
          <p>X: {x}, Y: {y}</p>
        </div>
      )}
    />
  );
}

// Alternative: children as a function
function DataProvider<T>({
  fetch,
  children
}: {
  fetch: () => Promise<T>;
  children: (data: T | null, loading: boolean) => ReactNode;
}) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch().then(d => {
      setData(d);
      setLoading(false);
    });
  }, [fetch]);

  return <>{children(data, loading)}</>;
}

// Usage
<DataProvider fetch={() => fetchUsers()}>
  {(users, loading) => 
    loading ? <Spinner /> : <UserList users={users} />
  }
</DataProvider>
```


---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*