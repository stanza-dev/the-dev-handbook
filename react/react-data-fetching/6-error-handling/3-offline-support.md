---
source_course: "react-data-fetching"
source_lesson: "react-data-fetching-offline-support"
---

# Offline Support

Make your app work without network connectivity.

## TanStack Query Offline Mode

```jsx
import { onlineManager } from '@tanstack/react-query';

// TanStack Query auto-pauses queries when offline
// And auto-resumes when back online

// Manually set online status (useful for testing)
onlineManager.setOnline(false);
```

## Persist Cache to Storage

```jsx
import { persistQueryClient } from '@tanstack/react-query-persist-client';
import { createSyncStoragePersister } from '@tanstack/query-sync-storage-persister';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      gcTime: 1000 * 60 * 60 * 24, // 24 hours
    },
  },
});

const persister = createSyncStoragePersister({
  storage: window.localStorage,
});

persistQueryClient({
  queryClient,
  persister,
  maxAge: 1000 * 60 * 60 * 24, // 24 hours
});
```

## Offline Indicator

```jsx
function OfflineIndicator() {
  const [isOnline, setIsOnline] = useState(navigator.onLine);

  useEffect(() => {
    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);
    
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  if (isOnline) return null;

  return (
    <div className="offline-banner">
      You're offline. Some features may be unavailable.
    </div>
  );
}
```

## Queue Mutations for Later

```jsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

function useQueuedMutation(mutationFn) {
  const queryClient = useQueryClient();
  const [queue, setQueue] = useState([]);

  const mutation = useMutation({
    mutationFn,
    networkMode: 'offlineFirst',
    onError: (error, variables) => {
      if (!navigator.onLine) {
        // Save to queue
        setQueue(q => [...q, variables]);
      }
    },
  });

  // Process queue when back online
  useEffect(() => {
    const handleOnline = async () => {
      for (const item of queue) {
        await mutation.mutateAsync(item);
      }
      setQueue([]);
    };

    window.addEventListener('online', handleOnline);
    return () => window.removeEventListener('online', handleOnline);
  }, [queue, mutation]);

  return mutation;
}
```

## Service Worker Caching

For complete offline support, combine with a service worker:

```jsx
// Register service worker
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js');
}

// sw.js
self.addEventListener('fetch', (event) => {
  if (event.request.url.includes('/api/')) {
    event.respondWith(
      caches.match(event.request)
        .then(response => response || fetch(event.request))
    );
  }
});
```

---

> 📘 *This lesson is part of the [React Data Fetching Patterns](https://stanza.dev/courses/react-data-fetching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*