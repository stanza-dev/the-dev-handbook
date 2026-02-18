---
source_course: "nextjs-testing"
source_lesson: "nextjs-testing-njs-database-mocking"
---

# Mocking Database Calls

For unit tests, mock your database client instead of hitting a real database.

## Mocking Prisma

```typescript
// __mocks__/@prisma/client.ts
const mockPrisma = {
  user: {
    findMany: jest.fn(),
    findUnique: jest.fn(),
    create: jest.fn(),
    update: jest.fn(),
    delete: jest.fn(),
  },
  // Add other models
};

export const PrismaClient = jest.fn(() => mockPrisma);
export { mockPrisma as prisma };
```

## Using in Tests

```typescript
import { prisma } from '@/lib/prisma';

jest.mock('@/lib/prisma');

const mockPrisma = prisma as jest.Mocked<typeof prisma>;

beforeEach(() => {
  jest.clearAllMocks();
});

it('fetches users', async () => {
  mockPrisma.user.findMany.mockResolvedValue([
    { id: '1', name: 'John', email: 'john@test.com' },
  ]);

  const users = await getUsers();

  expect(users).toHaveLength(1);
  expect(mockPrisma.user.findMany).toHaveBeenCalled();
});
```

## Integration Tests

For integration tests, use a test database (e.g., Docker) with real Prisma operations.

---

> 📘 *This lesson is part of the [Next.js Testing Strategies](https://stanza.dev/courses/nextjs-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*