---
source_course: "react-data-fetching"
source_lesson: "react-data-fetching-pagination-infinite"
---

# Pagination & Infinite Scroll

Handling large datasets efficiently.

## Basic Pagination

```jsx
function PaginatedTodos() {
  const [page, setPage] = useState(1);

  const { data, isLoading, isPlaceholderData } = useQuery({
    queryKey: ['todos', page],
    queryFn: () => fetchTodos(page),
    placeholderData: (previousData) => previousData,
  });

  return (
    <div>
      {isLoading ? (
        <Spinner />
      ) : (
        <>
          <TodoList todos={data.todos} />
          
          <div style={{ opacity: isPlaceholderData ? 0.5 : 1 }}>
            <button
              onClick={() => setPage(p => Math.max(1, p - 1))}
              disabled={page === 1}
            >
              Previous
            </button>
            <span>Page {page}</span>
            <button
              onClick={() => setPage(p => p + 1)}
              disabled={!data.hasMore}
            >
              Next
            </button>
          </div>
        </>
      )}
    </div>
  );
}
```

## Infinite Scroll with useInfiniteQuery

```jsx
import { useInfiniteQuery } from '@tanstack/react-query';

function InfiniteTodos() {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
    isLoading,
  } = useInfiniteQuery({
    queryKey: ['todos', 'infinite'],
    queryFn: ({ pageParam = 1 }) => fetchTodos(pageParam),
    getNextPageParam: (lastPage, pages) => {
      return lastPage.hasMore ? pages.length + 1 : undefined;
    },
  });

  if (isLoading) return <Spinner />;

  return (
    <div>
      {data.pages.map((page, i) => (
        <React.Fragment key={i}>
          {page.todos.map(todo => (
            <TodoItem key={todo.id} todo={todo} />
          ))}
        </React.Fragment>
      ))}
      
      <button
        onClick={() => fetchNextPage()}
        disabled={!hasNextPage || isFetchingNextPage}
      >
        {isFetchingNextPage
          ? 'Loading more...'
          : hasNextPage
          ? 'Load More'
          : 'Nothing more to load'}
      </button>
    </div>
  );
}
```

## Intersection Observer for Auto-Load

```jsx
function InfiniteTodos() {
  const { data, fetchNextPage, hasNextPage, isFetchingNextPage } = 
    useInfiniteQuery({ /* ... */ });

  const loadMoreRef = useRef();

  useEffect(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting && hasNextPage && !isFetchingNextPage) {
          fetchNextPage();
        }
      },
      { threshold: 1.0 }
    );

    if (loadMoreRef.current) {
      observer.observe(loadMoreRef.current);
    }

    return () => observer.disconnect();
  }, [fetchNextPage, hasNextPage, isFetchingNextPage]);

  return (
    <div>
      {/* Todo items... */}
      <div ref={loadMoreRef}>
        {isFetchingNextPage && <Spinner />}
      </div>
    </div>
  );
}
```

---

> 📘 *This lesson is part of the [React Data Fetching Patterns](https://stanza.dev/courses/react-data-fetching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*