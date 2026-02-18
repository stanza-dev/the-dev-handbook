---
source_course: "react-custom-hooks"
source_lesson: "react-custom-hooks-what-are-custom-hooks"
---

# What Are Custom Hooks?

Custom hooks extract reusable logic from components.

## The Basics

```jsx
// A custom hook is just a function that:
// 1. Starts with "use"
// 2. Can call other hooks

function useWindowWidth() {
  const [width, setWidth] = useState(window.innerWidth);
  
  useEffect(() => {
    const handleResize = () => setWidth(window.innerWidth);
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  return width;
}

// Usage
function Header() {
  const width = useWindowWidth();
  return width > 768 ? <DesktopNav /> : <MobileNav />;
}
```

## Why Custom Hooks?

### 1. Code Reuse

```jsx
// ❌ Without custom hook - duplicated logic
function ComponentA() {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetchData().then(setData).catch(setError).finally(() => setLoading(false));
  }, []);
  // ... 20 lines of fetch logic
}

function ComponentB() {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetchOther().then(setData).catch(setError).finally(() => setLoading(false));
  }, []);
  // ... same 20 lines duplicated!
}

// ✅ With custom hook - shared logic
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetch(url).then(r => r.json()).then(setData).catch(setError).finally(() => setLoading(false));
  }, [url]);
  
  return { data, loading, error };
}

function ComponentA() {
  const { data, loading, error } = useFetch('/api/data');
  // Clean component!
}
```

### 2. Separation of Concerns

```jsx
// Logic lives in hook, UI lives in component
function useAuth() {
  const [user, setUser] = useState(null);
  
  const login = async (credentials) => { /* ... */ };
  const logout = async () => { /* ... */ };
  
  return { user, login, logout };
}

function LoginPage() {
  const { login } = useAuth();
  // Only UI concerns here
  return <LoginForm onSubmit={login} />;
}
```

### 3. Easier Testing

```jsx
// Test the hook independently
import { renderHook, act } from '@testing-library/react';

test('useCounter increments', () => {
  const { result } = renderHook(() => useCounter());
  
  expect(result.current.count).toBe(0);
  
  act(() => {
    result.current.increment();
  });
  
  expect(result.current.count).toBe(1);
});
```

## Rules of Hooks (Apply to Custom Hooks Too)

1. **Only call at top level** - not in loops, conditions, nested functions
2. **Only call in React functions** - components or custom hooks
3. **Name starts with "use"** - enables linting and tooling

---

> 📘 *This lesson is part of the [Building Custom Hooks](https://stanza.dev/courses/react-custom-hooks) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*