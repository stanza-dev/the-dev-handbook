---
source_course: "react-data-fetching"
source_lesson: "react-data-fetching-realtime-data"
---

# Real-Time Data

Keeping data synchronized in real-time.

## Polling

Simplest approach - refetch on interval:

```jsx
useQuery({
  queryKey: ['notifications'],
  queryFn: fetchNotifications,
  refetchInterval: 30000, // Every 30 seconds
  refetchIntervalInBackground: false, // Pause when tab hidden
});
```

## Conditional Polling

```jsx
const { data } = useQuery({
  queryKey: ['order', orderId],
  queryFn: () => fetchOrder(orderId),
  refetchInterval: (query) => {
    // Poll while order is processing
    return query.state.data?.status === 'processing' ? 5000 : false;
  },
});
```

## WebSocket Integration

```jsx
import { useEffect } from 'react';
import { useQueryClient } from '@tanstack/react-query';

function useWebSocketUpdates() {
  const queryClient = useQueryClient();

  useEffect(() => {
    const ws = new WebSocket('wss://api.example.com/ws');

    ws.onmessage = (event) => {
      const message = JSON.parse(event.data);

      switch (message.type) {
        case 'TODO_CREATED':
          queryClient.invalidateQueries({ queryKey: ['todos'] });
          break;
        case 'TODO_UPDATED':
          queryClient.setQueryData(
            ['todo', message.data.id],
            message.data
          );
          break;
        case 'TODO_DELETED':
          queryClient.setQueryData(['todos'], (old) =>
            old?.filter(t => t.id !== message.data.id)
          );
          break;
      }
    };

    return () => ws.close();
  }, [queryClient]);
}
```

## Server-Sent Events (SSE)

```jsx
function useSSEUpdates(url) {
  const queryClient = useQueryClient();

  useEffect(() => {
    const eventSource = new EventSource(url);

    eventSource.onmessage = (event) => {
      const data = JSON.parse(event.data);
      queryClient.setQueryData(['realtime-data'], data);
    };

    eventSource.onerror = () => {
      eventSource.close();
    };

    return () => eventSource.close();
  }, [url, queryClient]);
}
```

## Optimistic + Real-Time

```jsx
function useTodoMutation() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: createTodo,
    onMutate: async (newTodo) => {
      // Optimistic update
      await queryClient.cancelQueries({ queryKey: ['todos'] });
      const previous = queryClient.getQueryData(['todos']);
      
      queryClient.setQueryData(['todos'], (old) => [
        ...old,
        { ...newTodo, id: 'temp-' + Date.now(), pending: true }
      ]);
      
      return { previous };
    },
    // Real-time WebSocket will provide the actual update
    // If it doesn't arrive, onSettled refetches
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });
}
```

---

> 📘 *This lesson is part of the [React Data Fetching Patterns](https://stanza.dev/courses/react-data-fetching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*