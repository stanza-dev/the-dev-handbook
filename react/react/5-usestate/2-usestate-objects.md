---
source_course: "react"
source_lesson: "react-usestate-objects"
---

# useState with Objects & Arrays

## Introduction

Many components need to manage complex state: a form with multiple fields, a list of items, or a nested data structure. When your state is an object or array, you must handle updates immutably — creating new references rather than modifying existing ones. This lesson explains why immutability matters and the patterns for updating objects and arrays in state.

## Key Concepts

- **Immutable Updates**: Creating a new object or array instead of modifying the existing one in place.
- **Spread Operator**: The `...` syntax for shallow-copying objects and arrays before applying changes.
- **Reference Equality**: React uses `Object.is()` to compare old and new state. If the reference is the same, React skips the re-render.
- **Nested Updates**: Updating a property deep inside a nested object requires spreading at every level.

## Real World Context

In a project management app, a task object might have `title`, `description`, `status`, and `assignee` fields. When the user changes the status, you need to update just that one field while preserving everything else. If you mutate the existing object, React will not detect the change and the UI will not update.

## Deep Dive

### Updating Objects

Always spread the previous state and override the changed property:

```tsx
function ProfileEditor() {
  const [profile, setProfile] = useState({
    name: 'Alice',
    email: 'alice@example.com',
    bio: '',
  });

  const updateField = (field: string, value: string) => {
    setProfile(prev => ({ ...prev, [field]: value }));
  };

  return (
    <form>
      <input value={profile.name} onChange={e => updateField('name', e.target.value)} />
      <input value={profile.email} onChange={e => updateField('email', e.target.value)} />
      <textarea value={profile.bio} onChange={e => updateField('bio', e.target.value)} />
    </form>
  );
}
```

### Updating Arrays

Common immutable array operations:

```tsx
function TaskManager() {
  const [tasks, setTasks] = useState([
    { id: 1, text: 'Review PR', done: false },
    { id: 2, text: 'Write tests', done: false },
  ]);

  // Add item
  const addTask = (text: string) => {
    setTasks(prev => [...prev, { id: Date.now(), text, done: false }]);
  };

  // Update item
  const toggleTask = (id: number) => {
    setTasks(prev => prev.map(task =>
      task.id === id ? { ...task, done: !task.done } : task
    ));
  };

  // Remove item
  const removeTask = (id: number) => {
    setTasks(prev => prev.filter(task => task.id !== id));
  };

  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          <span style={{ textDecoration: task.done ? 'line-through' : 'none' }}>
            {task.text}
          </span>
          <button onClick={() => toggleTask(task.id)}>Toggle</button>
          <button onClick={() => removeTask(task.id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
}
```

### Why Mutation Fails

```tsx
// This will NOT trigger a re-render
const handleBroken = () => {
  profile.name = 'Bob';     // Mutating the existing object
  setProfile(profile);       // Same reference — React skips update
};

// This WILL trigger a re-render
const handleCorrect = () => {
  setProfile({ ...profile, name: 'Bob' }); // New object reference
};
```

## Common Pitfalls

1. **Mutating state directly** — Methods like `Array.push()`, `Array.splice()`, `sort()` (in-place), and direct property assignment mutate the original. Use `concat`, `filter`, `map`, spread, and `toSorted()` instead.
2. **Shallow copying nested objects** — The spread operator only copies one level deep. For nested updates, you must spread at each level or consider using a helper library like Immer.

## Best Practices

1. **Use the functional form of setState** — `setTasks(prev => [...prev, newItem])` guarantees you are working with the latest state, which is especially important in event handlers that run multiple updates.
2. **Flatten deeply nested state** — If your state is deeply nested, consider normalizing it into flatter structures. This makes immutable updates simpler and less error-prone.

## Summary

- State objects and arrays must be updated immutably by creating new references with the spread operator.
- React uses reference equality to detect changes — mutating the existing reference will not trigger a re-render.
- Use `map`, `filter`, `concat`, and spread instead of `push`, `splice`, and direct assignment.

## Code Examples

**Immutable add, update, and delete operations on array state**

```tsx
function TodoApp() {
  const [todos, setTodos] = useState([
    { id: 1, text: 'Learn React', done: false },
  ]);

  const addTodo = (text: string) => {
    setTodos(prev => [...prev, { id: Date.now(), text, done: false }]);
  };

  const toggleTodo = (id: number) => {
    setTodos(prev => prev.map(todo =>
      todo.id === id ? { ...todo, done: !todo.done } : todo
    ));
  };

  const deleteTodo = (id: number) => {
    setTodos(prev => prev.filter(todo => todo.id !== id));
  };

  return <div>{/* render todos */}</div>;
}
```


## Resources

- [Updating Objects in State](https://react.dev/learn/updating-objects-in-state) — Official guide on immutable object state updates
- [Updating Arrays in State](https://react.dev/learn/updating-arrays-in-state) — Official guide on immutable array state updates

---

> 📘 *This lesson is part of the [React Basics](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*