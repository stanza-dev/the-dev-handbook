---
source_course: "react-data-fetching"
source_lesson: "react-data-fetching-managing-loading-states"
---

# Managing Loading & Error States

Good UX requires clear feedback about data loading status.

## The Three States

```jsx
function DataDisplay() {
  const [data, setData] = useState(null);
  const [status, setStatus] = useState('idle'); // idle | loading | success | error
  const [error, setError] = useState(null);

  // Derived state
  const isIdle = status === 'idle';
  const isLoading = status === 'loading';
  const isError = status === 'error';
  const isSuccess = status === 'success';

  if (isIdle) return <button onClick={fetchData}>Load Data</button>;
  if (isLoading) return <Spinner />;
  if (isError) return <ErrorMessage error={error} onRetry={fetchData} />;
  return <DataView data={data} />;
}
```

## Loading Indicators

### Spinner Component

```jsx
function Spinner({ size = 'medium' }) {
  return (
    <div 
      className={`spinner spinner-${size}`}
      role="status"
      aria-label="Loading"
    >
      <span className="sr-only">Loading...</span>
    </div>
  );
}
```

### Skeleton Loading

Show placeholder content that mimics the final layout:

```jsx
function UserCardSkeleton() {
  return (
    <div className="user-card">
      <div className="skeleton skeleton-avatar" />
      <div className="skeleton skeleton-text" />
      <div className="skeleton skeleton-text short" />
    </div>
  );
}

function UserList() {
  const { data, isLoading } = useFetchUsers();

  if (isLoading) {
    return (
      <div>
        {[1, 2, 3].map(i => <UserCardSkeleton key={i} />)}
      </div>
    );
  }

  return data.map(user => <UserCard key={user.id} user={user} />);
}
```

## Error Handling UI

```jsx
function ErrorMessage({ error, onRetry }) {
  return (
    <div className="error-container" role="alert">
      <h3>Something went wrong</h3>
      <p>{error.message}</p>
      {onRetry && (
        <button onClick={onRetry}>
          Try Again
        </button>
      )}
    </div>
  );
}
```

## Empty States

Don't forget the "no data" case:

```jsx
function SearchResults({ query, results }) {
  if (!query) {
    return <p>Enter a search term to begin</p>;
  }

  if (results.length === 0) {
    return (
      <div className="empty-state">
        <img src="/no-results.svg" alt="" />
        <p>No results found for "{query}"</p>
        <p>Try adjusting your search terms</p>
      </div>
    );
  }

  return (
    <ul>
      {results.map(result => (
        <li key={result.id}>{result.title}</li>
      ))}
    </ul>
  );
}
```

## Progressive Loading

Show partial data as it becomes available:

```jsx
function Dashboard() {
  const { user, isLoading: userLoading } = useUser();
  const { stats, isLoading: statsLoading } = useStats();
  const { activity, isLoading: activityLoading } = useActivity();

  return (
    <div>
      {userLoading ? <HeaderSkeleton /> : <Header user={user} />}
      {statsLoading ? <StatsSkeleton /> : <StatsPanel stats={stats} />}
      {activityLoading ? <ActivitySkeleton /> : <ActivityFeed activity={activity} />}
    </div>
  );
}
```

---

> 📘 *This lesson is part of the [React Data Fetching Patterns](https://stanza.dev/courses/react-data-fetching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*