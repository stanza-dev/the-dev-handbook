---
source_course: "react-components-patterns"
source_lesson: "react-components-patterns-render-props"
---

# Render Props: Sharing Logic via Functions

A render prop is a function prop that a component uses to know what to render. It's a technique for sharing code between components.

## The Pattern

```tsx
type MousePosition = { x: number; y: number };

type MouseTrackerProps = {
  render: (position: MousePosition) => ReactNode;
};

function MouseTracker({ render }: MouseTrackerProps) {
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

// Usage
<MouseTracker
  render={({ x, y }) => (
    <div>
      Mouse position: {x}, {y}
    </div>
  )}
/>
```

## Using Children as Render Prop

The `children` prop can also be a function:

```tsx
type MouseTrackerProps = {
  children: (position: MousePosition) => ReactNode;
};

function MouseTracker({ children }: MouseTrackerProps) {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  // ... same logic
  
  return <>{children(position)}</>;
}

// Usage - cleaner syntax
<MouseTracker>
  {({ x, y }) => (
    <div>Mouse: {x}, {y}</div>
  )}
</MouseTracker>
```

## Real-World Example: Data Fetching

```tsx
type FetchState<T> = {
  data: T | null;
  loading: boolean;
  error: Error | null;
};

type FetchProps<T> = {
  url: string;
  children: (state: FetchState<T>) => ReactNode;
};

function Fetch<T>({ url, children }: FetchProps<T>) {
  const [state, setState] = useState<FetchState<T>>({
    data: null,
    loading: true,
    error: null
  });
  
  useEffect(() => {
    let cancelled = false;
    
    fetch(url)
      .then(res => res.json())
      .then(data => {
        if (!cancelled) {
          setState({ data, loading: false, error: null });
        }
      })
      .catch(error => {
        if (!cancelled) {
          setState({ data: null, loading: false, error });
        }
      });
    
    return () => { cancelled = true; };
  }, [url]);
  
  return <>{children(state)}</>;
}

// Usage
<Fetch<User[]> url="/api/users">
  {({ data, loading, error }) => {
    if (loading) return <Spinner />;
    if (error) return <Error message={error.message} />;
    return <UserList users={data!} />;
  }}
</Fetch>
```

## Toggle Example

```tsx
type ToggleProps = {
  children: (props: {
    on: boolean;
    toggle: () => void;
    setOn: (value: boolean) => void;
  }) => ReactNode;
};

function Toggle({ children }: ToggleProps) {
  const [on, setOn] = useState(false);
  const toggle = () => setOn(prev => !prev);
  
  return <>{children({ on, toggle, setOn })}</>;
}

// Usage
<Toggle>
  {({ on, toggle }) => (
    <div>
      <button onClick={toggle}>
        {on ? 'ON' : 'OFF'}
      </button>
      {on && <Settings />}
    </div>
  )}
</Toggle>
```

## Render Props vs Custom Hooks

Custom Hooks have largely replaced render props for logic reuse:

```tsx
// Render Prop approach
<MouseTracker>
  {({ x, y }) => <Cursor x={x} y={y} />}
</MouseTracker>

// Custom Hook approach (usually preferred)
function Cursor() {
  const { x, y } = useMousePosition();
  return <div style={{ left: x, top: y }} />;
}
```

**When to still use Render Props:**
- When you need to pass data to JSX without creating a new component
- When integrating with class components
- When the render logic is simple and inline

## Resources

- [Render Props Pattern](https://react.dev/learn/passing-props-to-a-component) — React documentation on passing data to components

---

> 📘 *This lesson is part of the [React Components & Patterns](https://stanza.dev/courses/react-components-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*