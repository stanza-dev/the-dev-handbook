---
source_course: "nextjs-api-patterns"
source_lesson: "nextjs-api-patterns-njs-query-params"
---

# Query Parameters and Pagination

## Introduction

List endpoints need filtering, sorting, and pagination. Query parameters make these features possible without creating dozens of separate routes.

## Key Concepts

**Query parameters** are the `?key=value` pairs in URLs:

- **Filtering**: `?status=active&role=admin`
- **Sorting**: `?sort=createdAt&order=desc`
- **Pagination**: `?page=2&limit=20`
- **Searching**: `?q=john`

## Real World Context

Without pagination:
- Listing 10,000 users returns 10,000 records
- Response time degrades as data grows
- Mobile clients download megabytes of JSON
- Your database cries

## Deep Dive

### Parsing Query Parameters

```typescript
export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  
  // Get string value (or null)
  const search = searchParams.get('q');
  
  // Get number with default
  const page = parseInt(searchParams.get('page') || '1');
  const limit = Math.min(parseInt(searchParams.get('limit') || '10'), 100);
  
  // Get array (for multi-select filters)
  const statuses = searchParams.getAll('status');
  // ?status=active&status=pending → ['active', 'pending']
  
  return Response.json({ search, page, limit, statuses });
}
```

### Cursor-Based Pagination (Recommended)

```typescript
const CursorPaginationSchema = z.object({
  cursor: z.string().optional(),
  limit: z.coerce.number().min(1).max(100).default(20),
});

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const { cursor, limit } = CursorPaginationSchema.parse({
    cursor: searchParams.get('cursor'),
    limit: searchParams.get('limit'),
  });

  const items = await prisma.post.findMany({
    take: limit + 1, // Fetch one extra to check if there's more
    cursor: cursor ? { id: cursor } : undefined,
    orderBy: { createdAt: 'desc' },
  });

  const hasMore = items.length > limit;
  const data = hasMore ? items.slice(0, -1) : items;
  const nextCursor = hasMore ? data[data.length - 1].id : null;

  return Response.json({
    data,
    pagination: {
      nextCursor,
      hasMore,
    },
  });
}
```

### Offset Pagination

```typescript
const OffsetPaginationSchema = z.object({
  page: z.coerce.number().min(1).default(1),
  limit: z.coerce.number().min(1).max(100).default(20),
});

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const { page, limit } = OffsetPaginationSchema.parse({
    page: searchParams.get('page'),
    limit: searchParams.get('limit'),
  });

  const skip = (page - 1) * limit;

  const [items, total] = await Promise.all([
    prisma.post.findMany({ skip, take: limit }),
    prisma.post.count(),
  ]);

  return Response.json({
    data: items,
    pagination: {
      page,
      limit,
      total,
      totalPages: Math.ceil(total / limit),
    },
  });
}
```

### Filtering and Sorting

```typescript
const ListUsersSchema = z.object({
  role: z.enum(['user', 'admin', 'moderator']).optional(),
  status: z.enum(['active', 'inactive']).optional(),
  search: z.string().optional(),
  sortBy: z.enum(['name', 'createdAt', 'email']).default('createdAt'),
  order: z.enum(['asc', 'desc']).default('desc'),
});

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  
  const filters = ListUsersSchema.parse({
    role: searchParams.get('role'),
    status: searchParams.get('status'),
    search: searchParams.get('q'),
    sortBy: searchParams.get('sortBy'),
    order: searchParams.get('order'),
  });

  const where: Prisma.UserWhereInput = {};
  
  if (filters.role) where.role = filters.role;
  if (filters.status) where.status = filters.status;
  if (filters.search) {
    where.OR = [
      { name: { contains: filters.search, mode: 'insensitive' } },
      { email: { contains: filters.search, mode: 'insensitive' } },
    ];
  }

  const users = await prisma.user.findMany({
    where,
    orderBy: { [filters.sortBy]: filters.order },
  });

  return Response.json(users);
}
```

## Common Pitfalls

1. **No limit cap**: Always cap the limit to prevent `?limit=1000000` from crashing your server.

2. **Offset pagination at scale**: Offset pagination becomes slow with large datasets. Use cursor pagination for better performance.

3. **Not validating sort fields**: Allow sorting only by indexed columns to prevent slow queries.

## Best Practices

- **Use cursor pagination for infinite scroll**: Better performance, no skipped items
- **Use offset pagination for page numbers**: Users can jump to page 5 directly
- **Cap limits**: `Math.min(userLimit, 100)` prevents abuse
- **Validate sort columns**: Only allow sorting by indexed fields

## Summary

Query parameters enable filtering, sorting, and pagination. Parse them from `new URL(request.url).searchParams`, validate with Zod, and always cap limits. Choose cursor pagination for performance-critical endpoints and offset pagination when users need page numbers.

## Resources

- [Prisma Pagination](https://www.prisma.io/docs/concepts/components/prisma-client/pagination) — Prisma pagination patterns

---

> 📘 *This lesson is part of the [Next.js API Routes & Patterns](https://stanza.dev/courses/nextjs-api-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*