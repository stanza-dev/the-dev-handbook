---
source_course: "react-data-fetching"
source_lesson: "react-data-fetching-use-mutation-hook"
---

# useMutation Hook

For data modifications (POST, PUT, DELETE), we need different patterns than read queries.

## The Hook

```jsx
import { useState, useCallback } from 'react';

function useMutation(mutationFn, options = {}) {
  const { onSuccess, onError, onSettled } = options;
  
  const [status, setStatus] = useState('idle');
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);

  const mutate = useCallback(
    async (variables) => {
      setStatus('loading');
      setError(null);

      try {
        const result = await mutationFn(variables);
        setData(result);
        setStatus('success');
        onSuccess?.(result, variables);
        return result;
      } catch (err) {
        setError(err);
        setStatus('error');
        onError?.(err, variables);
        throw err;
      } finally {
        onSettled?.(data, error, variables);
      }
    },
    [mutationFn, onSuccess, onError, onSettled]
  );

  const reset = useCallback(() => {
    setStatus('idle');
    setData(null);
    setError(null);
  }, []);

  return {
    mutate,
    reset,
    status,
    data,
    error,
    isIdle: status === 'idle',
    isLoading: status === 'loading',
    isSuccess: status === 'success',
    isError: status === 'error',
  };
}
```

## Usage: Create Todo

```jsx
const createTodo = async (todo) => {
  const response = await fetch('/api/todos', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(todo),
  });
  return response.json();
};

function AddTodo({ onTodoAdded }) {
  const [text, setText] = useState('');
  
  const { mutate, isLoading, isError, error, reset } = useMutation(
    createTodo,
    {
      onSuccess: (newTodo) => {
        setText(''); // Clear input
        onTodoAdded(newTodo); // Notify parent
      },
    }
  );

  const handleSubmit = (e) => {
    e.preventDefault();
    mutate({ text, completed: false });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={text}
        onChange={(e) => setText(e.target.value)}
        placeholder="New todo"
        disabled={isLoading}
      />
      <button type="submit" disabled={isLoading}>
        {isLoading ? 'Adding...' : 'Add'}
      </button>
      {isError && (
        <div className="error">
          {error.message}
          <button type="button" onClick={reset}>Dismiss</button>
        </div>
      )}
    </form>
  );
}
```

## Usage: Delete Item

```jsx
const deleteItem = async (id) => {
  await fetch(`/api/items/${id}`, { method: 'DELETE' });
  return id;
};

function ItemList({ items, onItemDeleted }) {
  const { mutate: deleteMutation, isLoading } = useMutation(
    deleteItem,
    { onSuccess: onItemDeleted }
  );

  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>
          {item.name}
          <button
            onClick={() => deleteMutation(item.id)}
            disabled={isLoading}
          >
            Delete
          </button>
        </li>
      ))}
    </ul>
  );
}
```

---

> 📘 *This lesson is part of the [React Data Fetching Patterns](https://stanza.dev/courses/react-data-fetching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*