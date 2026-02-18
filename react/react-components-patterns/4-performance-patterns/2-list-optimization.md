---
source_course: "react-components-patterns"
source_lesson: "react-components-patterns-list-optimization"
---

# Optimizing List Rendering

Lists are a common performance bottleneck. Learn patterns to keep them fast.

## The Key Prop

Keys help React identify which items changed:

```tsx
// ❌ Bad - index as key causes issues with reordering
{items.map((item, index) => (
  <Item key={index} {...item} />
))}

// ✅ Good - stable unique ID
{items.map(item => (
  <Item key={item.id} {...item} />
))}
```

## Memoize List Items

```tsx
const ListItem = memo(function ListItem({ 
  item, 
  onSelect, 
  onDelete 
}: ListItemProps) {
  return (
    <div className="list-item">
      <span>{item.name}</span>
      <button onClick={() => onSelect(item.id)}>Select</button>
      <button onClick={() => onDelete(item.id)}>Delete</button>
    </div>
  );
});

function List({ items }: { items: Item[] }) {
  // Stable callbacks
  const handleSelect = useCallback((id: string) => {
    console.log('Selected:', id);
  }, []);
  
  const handleDelete = useCallback((id: string) => {
    setItems(prev => prev.filter(item => item.id !== id));
  }, []);
  
  return (
    <div>
      {items.map(item => (
        <ListItem
          key={item.id}
          item={item}
          onSelect={handleSelect}
          onDelete={handleDelete}
        />
      ))}
    </div>
  );
}
```

## Virtualization for Long Lists

For lists with hundreds+ items, render only visible items:

```tsx
import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualList({ items }: { items: Item[] }) {
  const parentRef = useRef<HTMLDivElement>(null);
  
  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50, // Estimated row height
  });
  
  return (
    <div ref={parentRef} style={{ height: 400, overflow: 'auto' }}>
      <div
        style={{
          height: virtualizer.getTotalSize(),
          position: 'relative'
        }}
      >
        {virtualizer.getVirtualItems().map(virtualRow => (
          <div
            key={virtualRow.key}
            style={{
              position: 'absolute',
              top: virtualRow.start,
              height: virtualRow.size,
              width: '100%'
            }}
          >
            <ListItem item={items[virtualRow.index]} />
          </div>
        ))}
      </div>
    </div>
  );
}
```

## Pagination vs Infinite Scroll

```tsx
// Pagination - load fixed pages
function PaginatedList() {
  const [page, setPage] = useState(1);
  const { data, isLoading } = useQuery({
    queryKey: ['items', page],
    queryFn: () => fetchItems(page)
  });
  
  return (
    <div>
      {data?.items.map(item => <Item key={item.id} {...item} />)}
      <Pagination 
        page={page} 
        total={data?.totalPages} 
        onChange={setPage} 
      />
    </div>
  );
}

// Infinite scroll - load more as user scrolls
function InfiniteList() {
  const { data, fetchNextPage, hasNextPage } = useInfiniteQuery({
    queryKey: ['items'],
    queryFn: ({ pageParam = 1 }) => fetchItems(pageParam),
    getNextPageParam: (lastPage) => lastPage.nextPage
  });
  
  const items = data?.pages.flatMap(page => page.items) ?? [];
  
  return (
    <div>
      {items.map(item => <Item key={item.id} {...item} />)}
      {hasNextPage && (
        <button onClick={() => fetchNextPage()}>Load More</button>
      )}
    </div>
  );
}
```

## Avoiding Re-renders on Filter/Sort

```tsx
function FilterableList({ items }: { items: Item[] }) {
  const [filter, setFilter] = useState('');
  const [sortBy, setSortBy] = useState<'name' | 'date'>('name');
  
  // Memoize filtered/sorted list
  const displayItems = useMemo(() => {
    let result = items;
    
    if (filter) {
      result = result.filter(item => 
        item.name.toLowerCase().includes(filter.toLowerCase())
      );
    }
    
    result = [...result].sort((a, b) => {
      if (sortBy === 'name') return a.name.localeCompare(b.name);
      return new Date(b.date).getTime() - new Date(a.date).getTime();
    });
    
    return result;
  }, [items, filter, sortBy]);
  
  return (
    <div>
      <input value={filter} onChange={e => setFilter(e.target.value)} />
      <select value={sortBy} onChange={e => setSortBy(e.target.value)}>
        <option value="name">Name</option>
        <option value="date">Date</option>
      </select>
      {displayItems.map(item => (
        <MemoizedItem key={item.id} item={item} />
      ))}
    </div>
  );
}
```

## Resources

- [Rendering Lists](https://react.dev/learn/rendering-lists) — Official React guide on rendering lists

---

> 📘 *This lesson is part of the [React Components & Patterns](https://stanza.dev/courses/react-components-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*