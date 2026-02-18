---
source_course: "react-state-architecture"
source_lesson: "react-state-architecture-state-synchronization"
---

# State Synchronization Patterns

Keeping client and server state in sync.

## Optimistic Updates

```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

function useTodoToggle() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async (todoId: string) => {
      const response = await fetch(`/api/todos/${todoId}/toggle`, {
        method: 'PATCH',
      });
      return response.json();
    },
    onMutate: async (todoId) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['todos'] });

      // Snapshot current data
      const previousTodos = queryClient.getQueryData(['todos']);

      // Optimistically update
      queryClient.setQueryData(['todos'], (old: Todo[]) =>
        old.map((todo) =>
          todo.id === todoId
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      );

      // Return context for rollback
      return { previousTodos };
    },
    onError: (err, todoId, context) => {
      // Rollback on error
      queryClient.setQueryData(['todos'], context?.previousTodos);
    },
    onSettled: () => {
      // Refetch to ensure sync
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });
}
```

## Real-time Synchronization

```tsx
import { useEffect } from 'react';
import { useQueryClient } from '@tanstack/react-query';

function useRealtimeTodos() {
  const queryClient = useQueryClient();

  useEffect(() => {
    const ws = new WebSocket('wss://api.example.com/todos');

    ws.onmessage = (event) => {
      const message = JSON.parse(event.data);

      switch (message.type) {
        case 'TODO_ADDED':
          queryClient.setQueryData(['todos'], (old: Todo[]) => [
            ...old,
            message.payload,
          ]);
          break;
        case 'TODO_UPDATED':
          queryClient.setQueryData(['todos'], (old: Todo[]) =>
            old.map((todo) =>
              todo.id === message.payload.id ? message.payload : todo
            )
          );
          break;
        case 'TODO_DELETED':
          queryClient.setQueryData(['todos'], (old: Todo[]) =>
            old.filter((todo) => todo.id !== message.payload.id)
          );
          break;
      }
    };

    return () => ws.close();
  }, [queryClient]);
}
```

## Cross-Tab Synchronization

```tsx
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';

const useAuthStore = create(
  persist(
    (set) => ({
      user: null,
      setUser: (user) => set({ user }),
      logout: () => set({ user: null }),
    }),
    {
      name: 'auth-storage',
      storage: createJSONStorage(() => localStorage),
    }
  )
);

// Listen for storage events (cross-tab)
function useCrossTabSync() {
  useEffect(() => {
    const handleStorageChange = (e: StorageEvent) => {
      if (e.key === 'auth-storage') {
        // Zustand persist handles this automatically
        // But for custom logic:
        const newState = JSON.parse(e.newValue || '{}');
        useAuthStore.setState(newState.state);
      }
    };

    window.addEventListener('storage', handleStorageChange);
    return () => window.removeEventListener('storage', handleStorageChange);
  }, []);
}
```

## Conflict Resolution

```tsx
type Version = number;
type ServerData<T> = { data: T; version: Version };

function useOptimisticUpdate<T>(queryKey: string[]) {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async ({
      data,
      expectedVersion,
    }: {
      data: T;
      expectedVersion: Version;
    }) => {
      const response = await fetch('/api/data', {
        method: 'PUT',
        body: JSON.stringify({ data, expectedVersion }),
      });

      if (response.status === 409) {
        throw new Error('CONFLICT');
      }

      return response.json();
    },
    onError: (error) => {
      if (error.message === 'CONFLICT') {
        // Refetch and let user resolve
        queryClient.invalidateQueries({ queryKey });
      }
    },
  });
}
```

---

> 📘 *This lesson is part of the [React State Architecture](https://stanza.dev/courses/react-state-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*