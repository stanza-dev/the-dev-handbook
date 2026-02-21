---
source_course: "react-intermediate"
source_lesson: "react-query-mutations"
---

# Mutations and Cache Invalidation

## Introduction

While queries read data from the server, mutations write data. TanStack Query's useMutation hook provides a structured way to handle create, update, and delete operations, including optimistic updates and automatic cache invalidation.

## Key Concepts

- **Mutation**: An operation that changes server-side data (POST, PUT, PATCH, DELETE requests).
- **Cache invalidation**: Marking cached queries as stale so they refetch fresh data after a mutation.
- **Optimistic update**: Updating the UI immediately before the server confirms the change, providing an instant feel.

## Real World Context

In a task management app, when a user marks a task as complete, you want the checkbox to update instantly (optimistic update), send the PATCH request to the server, and then either confirm the update or roll back if it fails. Additionally, the task list and task count queries should refetch to reflect the change.

## Deep Dive

The \`useMutation\` hook accepts a mutation function and lifecycle callbacks. The most important callbacks are \`onSuccess\` for cache invalidation and \`onMutate\` / \`onError\` for optimistic updates.

\`\`\`tsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

function AddTodoButton() {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: (newTodo: { title: string }) =>
      fetch('/api/todos', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(newTodo),
      }).then(res => res.json()),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });

  return (
    <button
      onClick={() => mutation.mutate({ title: 'New Todo' })}
      disabled={mutation.isPending}
    >
      {mutation.isPending ? 'Adding...' : 'Add Todo'}
    </button>
  );
}
\`\`\`

For optimistic updates, you manually update the cache before the server responds and roll back on error:

\`\`\`tsx
const toggleMutation = useMutation({
  mutationFn: (id: string) =>
    fetch(\`/api/todos/\${id}/toggle\`, { method: 'PATCH' }),
  onMutate: async (id) => {
    await queryClient.cancelQueries({ queryKey: ['todos'] });
    const previous = queryClient.getQueryData(['todos']);
    queryClient.setQueryData(['todos'], (old) =>
      old.map(t => t.id === id ? { ...t, done: !t.done } : t)
    );
    return { previous };
  },
  onError: (err, id, context) => {
    queryClient.setQueryData(['todos'], context.previous);
  },
  onSettled: () => {
    queryClient.invalidateQueries({ queryKey: ['todos'] });
  },
});
\`\`\`

## Common Pitfalls

1. **Forgetting to invalidate after mutation** — The UI shows stale data because the query cache was not updated. Always invalidate related queries in onSuccess.
2. **Not handling rollback in optimistic updates** — If the mutation fails, the UI is stuck in the optimistic state. Always implement onError rollback.

## Best Practices

1. **Use onSettled for invalidation in optimistic updates** — This ensures the cache is synced whether the mutation succeeds or fails.
2. **Use mutation.isPending for UI feedback** — Disable buttons and show loading indicators during the mutation to prevent double-submission.

## Summary

- useMutation handles server-side write operations with lifecycle callbacks.
- Invalidate related queries in onSuccess to keep the cache fresh.
- Optimistic updates provide instant feedback but require onError rollback.

## Code Examples

**Mutation with automatic cache invalidation**

```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

function AddTodo() {
  const queryClient = useQueryClient();
  const mutation = useMutation({
    mutationFn: (title: string) =>
      fetch('/api/todos', {
        method: 'POST',
        body: JSON.stringify({ title }),
        headers: { 'Content-Type': 'application/json' },
      }).then(r => r.json()),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });

  return (
    <button
      onClick={() => mutation.mutate('New task')}
      disabled={mutation.isPending}
    >
      {mutation.isPending ? 'Adding...' : 'Add Todo'}
    </button>
  );
}
```


## Resources

- [Mutations](https://tanstack.com/query/latest/docs/framework/react/guides/mutations) — Official TanStack Query mutations guide

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*