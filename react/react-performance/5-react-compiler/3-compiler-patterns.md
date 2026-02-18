---
source_course: "react-performance"
source_lesson: "react-performance-compiler-patterns"
---

# Writing Compiler-Friendly Code

Patterns that work well with React Compiler.

## Immutable Updates

```jsx
// ✅ Compiler-friendly: Immutable
function TodoList() {
  const [todos, setTodos] = useState([]);
  
  const addTodo = (text) => {
    setTodos([...todos, { id: Date.now(), text }]);
  };
  
  const toggleTodo = (id) => {
    setTodos(todos.map(todo =>
      todo.id === id ? { ...todo, done: !todo.done } : todo
    ));
  };
}

// ❌ Compiler can't optimize: Mutation
function BadTodoList() {
  const [todos, setTodos] = useState([]);
  
  const addTodo = (text) => {
    todos.push({ id: Date.now(), text }); // Mutation!
    setTodos(todos);
  };
}
```

## Pure Calculations

```jsx
// ✅ Pure - compiler can cache
function formatPrice(price, currency) {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency,
  }).format(price);
}

function PriceDisplay({ price, currency }) {
  const formatted = formatPrice(price, currency);
  return <span>{formatted}</span>;
}

// ❌ Impure - side effect
function formatPrice(price) {
  console.log('Formatting:', price); // Side effect!
  return `$${price.toFixed(2)}`;
}
```

## Component Structure

```jsx
// ✅ Compiler-friendly: Clear data flow
function UserProfile({ userId }) {
  const user = useUser(userId);
  const posts = usePosts(userId);
  
  if (!user) return <Loading />;
  
  return (
    <div>
      <UserHeader user={user} />
      <PostList posts={posts} />
    </div>
  );
}

// ❌ Less optimal: Conditional hooks
function UserProfile({ userId }) {
  if (!userId) return null; // Early return before hooks!
  
  const user = useUser(userId);
  // Hooks must be called unconditionally
}
```

## Derived State

```jsx
// ✅ Compiler auto-memoizes derived values
function FilteredList({ items, filter }) {
  // Compiler detects this depends on items and filter
  const filteredItems = items.filter(item =>
    item.name.toLowerCase().includes(filter.toLowerCase())
  );
  
  return (
    <ul>
      {filteredItems.map(item => (
        <ListItem key={item.id} item={item} />
      ))}
    </ul>
  );
}
```

## Event Handlers

```jsx
// ✅ Compiler memoizes these automatically
function Counter() {
  const [count, setCount] = useState(0);
  
  const increment = () => setCount(c => c + 1);
  const decrement = () => setCount(c => c - 1);
  
  return (
    <div>
      <Button onClick={decrement}>-</Button>
      <span>{count}</span>
      <Button onClick={increment}>+</Button>
    </div>
  );
}
```

## Refs for Escape Hatch

```jsx
// ✅ Refs bypass reactivity when needed
function Stopwatch() {
  const [time, setTime] = useState(0);
  const intervalRef = useRef(null);
  
  const start = () => {
    intervalRef.current = setInterval(() => {
      setTime(t => t + 1);
    }, 1000);
  };
  
  const stop = () => {
    clearInterval(intervalRef.current);
  };
  
  return (
    <div>
      <span>{time}s</span>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
    </div>
  );
}
```

---

> 📘 *This lesson is part of the [React Performance Deep Dive](https://stanza.dev/courses/react-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*