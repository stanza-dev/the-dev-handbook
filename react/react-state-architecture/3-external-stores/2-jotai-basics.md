---
source_course: "react-state-architecture"
source_lesson: "react-state-architecture-jotai-basics"
---

# Jotai: Atomic State Management

Jotai uses atoms for fine-grained reactivity.

## Installation

```bash
npm install jotai
```

## Primitive Atoms

```jsx
import { atom, useAtom, useAtomValue, useSetAtom } from 'jotai';

// Create atoms
const countAtom = atom(0);
const nameAtom = atom('Guest');

// Use in components
function Counter() {
  const [count, setCount] = useAtom(countAtom);
  return (
    <button onClick={() => setCount(c => c + 1)}>
      Count: {count}
    </button>
  );
}

// Read-only
function CountDisplay() {
  const count = useAtomValue(countAtom);
  return <span>{count}</span>;
}

// Write-only
function IncrementButton() {
  const setCount = useSetAtom(countAtom);
  return <button onClick={() => setCount(c => c + 1)}>+</button>;
}
```

## Derived Atoms

```jsx
const countAtom = atom(0);
const doubleCountAtom = atom((get) => get(countAtom) * 2);

// Read-write derived atom
const countInDollarsAtom = atom(
  (get) => `$${get(countAtom).toFixed(2)}`,
  (get, set, newValue) => {
    const number = parseFloat(newValue.replace('$', ''));
    set(countAtom, number);
  }
);
```

## Async Atoms

```jsx
const userIdAtom = atom(1);

const userAtom = atom(async (get) => {
  const id = get(userIdAtom);
  const response = await fetch(`/api/users/${id}`);
  return response.json();
});

// Using with Suspense
function UserProfile() {
  const user = useAtomValue(userAtom);
  return <div>{user.name}</div>;
}

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <UserProfile />
    </Suspense>
  );
}
```

## Atom Families

```jsx
import { atomFamily } from 'jotai/utils';

// Create atoms dynamically
const todoAtomFamily = atomFamily((id) => atom(null));

function TodoItem({ id }) {
  const [todo, setTodo] = useAtom(todoAtomFamily(id));
  // ...
}
```

## Writable Derived Atoms (Actions)

```jsx
const todosAtom = atom([]);

// Action atom (write-only)
const addTodoAtom = atom(
  null, // read returns null
  (get, set, newTodo) => {
    const todos = get(todosAtom);
    set(todosAtom, [...todos, { id: Date.now(), ...newTodo }]);
  }
);

const removeTodoAtom = atom(
  null,
  (get, set, id) => {
    const todos = get(todosAtom);
    set(todosAtom, todos.filter(t => t.id !== id));
  }
);

// Usage
function AddTodoForm() {
  const addTodo = useSetAtom(addTodoAtom);
  
  const handleSubmit = (e) => {
    e.preventDefault();
    addTodo({ title: e.target.title.value });
  };
  
  return <form onSubmit={handleSubmit}>...</form>;
}
```

## Storage Persistence

```jsx
import { atomWithStorage } from 'jotai/utils';

const themeAtom = atomWithStorage('theme', 'light');
const userPrefsAtom = atomWithStorage('prefs', { notifications: true });
```

## Jotai vs Zustand

| Feature | Zustand | Jotai |
|---------|---------|-------|
| Mental model | Single store | Multiple atoms |
| Boilerplate | More | Less |
| Derived state | Manual | Built-in |
| Code splitting | Slices | Atoms |
| Async | Manual | Built-in |
| DevTools | Middleware | Plugin |

---

> 📘 *This lesson is part of the [React State Architecture](https://stanza.dev/courses/react-state-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*