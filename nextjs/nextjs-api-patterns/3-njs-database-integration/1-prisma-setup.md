---
source_course: "nextjs-api-patterns"
source_lesson: "nextjs-api-patterns-njs-prisma-setup"
---

# Database with Prisma

## Introduction

Most APIs need a database. Prisma is a type-safe ORM that makes database operations feel like TypeScript—autocomplete, type checking, and no raw SQL strings.

## Key Concepts

**Prisma** consists of three parts:

- **Schema**: Define your data models in `schema.prisma`
- **Client**: Auto-generated, type-safe database client
- **Migrate**: Database migration tool

## Real World Context

Prisma gives you:
- TypeScript types generated from your schema
- Autocomplete for queries and relations
- Protection against SQL injection
- Database-agnostic queries

## Deep Dive

### Setup

```bash
npm install prisma @prisma/client
npx prisma init
```

### Define Your Schema

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  role      String   @default("user")
  posts     Post[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Post {
  id        String   @id @default(cuid())
  title     String
  content   String?
  published Boolean  @default(false)
  author    User     @relation(fields: [authorId], references: [id])
  authorId  String
  createdAt DateTime @default(now())
}
```

### Generate Client and Migrate

```bash
npx prisma migrate dev --name init
npx prisma generate
```

### Singleton Pattern (Critical for Next.js)

```typescript
// lib/prisma.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma = globalForPrisma.prisma ?? new PrismaClient({
  log: process.env.NODE_ENV === 'development' ? ['query'] : [],
});

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}
```

### Using in Route Handlers

```typescript
import { prisma } from '@/lib/prisma';

export async function GET() {
  const users = await prisma.user.findMany({
    include: { posts: true },
    orderBy: { createdAt: 'desc' },
  });
  return Response.json(users);
}

export async function POST(request: Request) {
  const { name, email } = await request.json();
  
  const user = await prisma.user.create({
    data: { name, email },
  });
  
  return Response.json(user, { status: 201 });
}
```

### Common Query Patterns

```typescript
// Find with conditions
const activeUsers = await prisma.user.findMany({
  where: { role: 'admin' },
});

// Find one by unique field
const user = await prisma.user.findUnique({
  where: { email: 'john@example.com' },
});

// Find with relations
const userWithPosts = await prisma.user.findUnique({
  where: { id: userId },
  include: { posts: true },
});

// Select specific fields
const emails = await prisma.user.findMany({
  select: { email: true, name: true },
});
```

## Common Pitfalls

1. **Multiple PrismaClient instances**: Without the singleton pattern, hot reload creates new connections until your database runs out.

2. **Not handling missing records**: `findUnique` returns `null` if not found. Always check before using.

3. **Forgetting to generate**: After schema changes, run `npx prisma generate` before using new fields.

## Best Practices

- **Always use the singleton pattern**: Prevents connection exhaustion in development
- **Use transactions for related operations**: Keep data consistent
- **Index frequently queried fields**: Add `@@index` to your schema
- **Use `select` for large tables**: Don't fetch fields you don't need

## Summary

Prisma provides type-safe database access for your Next.js API. Define models in schema.prisma, generate the client, and use the singleton pattern to prevent connection issues. The generated client gives you autocomplete and type safety for all database operations.

## Resources

- [Prisma Documentation](https://www.prisma.io/docs) — Official Prisma documentation

---

> 📘 *This lesson is part of the [Next.js API Routes & Patterns](https://stanza.dev/courses/nextjs-api-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*