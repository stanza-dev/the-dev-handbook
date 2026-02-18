---
source_course: "react-performance"
source_lesson: "react-performance-memory-optimization"
---

# Memory Optimization Techniques

Reduce memory footprint for better performance.

## Cleaning Up Closures

```jsx
// ❌ Closure holds reference to large data
function DataProcessor({ data }) {
  const processData = useCallback(() => {
    // data (potentially large) captured in closure
    return data.map(item => transform(item));
  }, [data]);
  
  return <ProcessButton onClick={processData} />;
}

// ✅ Use refs to avoid closure capture
function DataProcessor({ data }) {
  const dataRef = useRef(data);
  dataRef.current = data;
  
  const processData = useCallback(() => {
    return dataRef.current.map(item => transform(item));
  }, []); // No data dependency
  
  return <ProcessButton onClick={processData} />;
}
```

## Normalizing Data

```jsx
// ❌ Denormalized - duplicated data
const posts = [
  {
    id: 1,
    title: 'Hello',
    author: { id: 1, name: 'Alice', avatar: '...' }, // Duplicated
  },
  {
    id: 2,
    title: 'World',
    author: { id: 1, name: 'Alice', avatar: '...' }, // Same author, duplicated
  },
];

// ✅ Normalized - single source
const state = {
  posts: {
    1: { id: 1, title: 'Hello', authorId: 1 },
    2: { id: 2, title: 'World', authorId: 1 },
  },
  users: {
    1: { id: 1, name: 'Alice', avatar: '...' },
  },
};
```

## Lazy Initialization

```jsx
// ❌ Expensive initial value computed every render
function SearchResults() {
  const [results, setResults] = useState(
    expensiveComputation() // Runs every render!
  );
}

// ✅ Lazy initialization - runs only once
function SearchResults() {
  const [results, setResults] = useState(() => {
    return expensiveComputation();
  });
}
```

## WeakMap for Associated Data

```jsx
// ✅ WeakMap allows garbage collection
const componentData = new WeakMap();

function useComponentMetadata(ref) {
  useEffect(() => {
    if (ref.current) {
      componentData.set(ref.current, {
        mountTime: Date.now(),
        renderCount: 0,
      });
    }
    
    return () => {
      // WeakMap entry is automatically garbage collected
      // when ref.current is no longer referenced
    };
  }, [ref]);
}
```

## Pagination and Windowing

```jsx
// ❌ Loading all data into memory
function AllUsers() {
  const { data: users } = useQuery('users', () =>
    fetch('/api/users').then(r => r.json()) // 10,000 users!
  );
  
  return <List items={users} />;
}

// ✅ Paginated - limited memory usage
function PaginatedUsers() {
  const [page, setPage] = useState(1);
  const { data } = useQuery(['users', page], () =>
    fetch(`/api/users?page=${page}&limit=50`).then(r => r.json())
  );
  
  return (
    <>
      <List items={data.users} />
      <Pagination page={page} onChange={setPage} />
    </>
  );
}
```

## Object Pooling

```jsx
// For frequently created/destroyed objects
class ObjectPool {
  constructor(factory, reset, initialSize = 10) {
    this.factory = factory;
    this.reset = reset;
    this.pool = Array.from({ length: initialSize }, factory);
  }
  
  acquire() {
    return this.pool.pop() || this.factory();
  }
  
  release(obj) {
    this.reset(obj);
    this.pool.push(obj);
  }
}

const particlePool = new ObjectPool(
  () => ({ x: 0, y: 0, vx: 0, vy: 0 }),
  (p) => { p.x = p.y = p.vx = p.vy = 0; }
);
```

---

> 📘 *This lesson is part of the [React Performance Deep Dive](https://stanza.dev/courses/react-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*