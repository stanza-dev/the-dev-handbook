---
source_course: "nextjs-optimization"
source_lesson: "nextjs-optimization-njs-code-splitting"
---

# Automatic and Manual Code Splitting

Next.js automatically code-splits your application, but you can optimize further.

## Automatic Splitting

- Each route gets its own bundle.
- Shared code is split into a separate chunk.
- Third-party libraries are grouped together.

## Dynamic Imports

Load components only when needed:

```typescript
import dynamic from 'next/dynamic';

const HeavyChart = dynamic(() => import('@/components/HeavyChart'), {
  loading: () => <p>Loading chart...</p>,
  ssr: false, // Don't render on server (for client-only libs)
});

export default function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      <HeavyChart />
    </div>
  );
}
```

## Lazy Loading Libraries

```typescript
// Heavy library loaded only when needed
const handleExport = async () => {
  const xlsx = await import('xlsx');
  xlsx.writeFile(workbook, 'data.xlsx');
};
```

## Client-Only Components

For components that can't be server-rendered (e.g., using `window`):

```typescript
const MapComponent = dynamic(() => import('./Map'), {
  ssr: false,
});
```

---

> 📘 *This lesson is part of the [Next.js Performance & Optimization](https://stanza.dev/courses/nextjs-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*