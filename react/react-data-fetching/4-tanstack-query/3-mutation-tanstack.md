---
source_course: "react-data-fetching"
source_lesson: "react-data-fetching-use-mutation-tanstack"
---

# useMutation for Data Changes

`useMutation` handles creating, updating, and deleting data.

## Basic Usage

```jsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

function AddTodo() {
  const queryClient = useQueryClient();
  
  const mutation = useMutation({
    mutationFn: (newTodo) => {
      return fetch('/api/todos', {
        method: 'POST',
        body: JSON.stringify(newTodo),
      }).then(res => res.json());
    },
    onSuccess: () => {
      // Invalidate and refetch
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });

  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      mutation.mutate({ title: 'New Todo' });
    }}>
      <button disabled={mutation.isPending}>
        {mutation.isPending ? 'Adding...' : 'Add Todo'}
      </button>
    </form>
  );
}
```

## Mutation Lifecycle

```jsx
const mutation = useMutation({
  mutationFn: updateUser,
  
  // Before mutation
  onMutate: async (variables) => {
    console.log('Starting mutation with:', variables);
    // Return context for rollback
    return { previousData: 'backup' };
  },
  
  // On success
  onSuccess: (data, variables, context) => {
    console.log('Success!', data);
    queryClient.invalidateQueries(['users']);
  },
  
  // On error
  onError: (error, variables, context) => {
    console.log('Error:', error.message);
    // Rollback using context
  },
  
  // Always runs (success or error)
  onSettled: (data, error, variables, context) => {
    console.log('Mutation finished');
  },
});
```

## Optimistic Updates

```jsx
const updateTodoMutation = useMutation({
  mutationFn: updateTodo,
  
  onMutate: async (updatedTodo) => {
    // Cancel outgoing refetches
    await queryClient.cancelQueries({ queryKey: ['todos'] });
    
    // Snapshot current data
    const previousTodos = queryClient.getQueryData(['todos']);
    
    // Optimistically update cache
    queryClient.setQueryData(['todos'], (old) =>
      old.map(todo =>
        todo.id === updatedTodo.id ? updatedTodo : todo
      )
    );
    
    // Return snapshot for rollback
    return { previousTodos };
  },
  
  onError: (err, updatedTodo, context) => {
    // Rollback to snapshot
    queryClient.setQueryData(['todos'], context.previousTodos);
  },
  
  onSettled: () => {
    // Refetch to ensure sync
    queryClient.invalidateQueries({ queryKey: ['todos'] });
  },
});
```

## Return Values

```jsx
const {
  mutate,      // Trigger mutation
  mutateAsync, // Trigger and return promise
  isPending,   // Currently mutating
  isSuccess,   // Mutation succeeded
  isError,     // Mutation failed
  error,       // Error object
  data,        // Returned data
  reset,       // Reset state
} = useMutation({ mutationFn });
```

---

> 📘 *This lesson is part of the [React Data Fetching Patterns](https://stanza.dev/courses/react-data-fetching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*