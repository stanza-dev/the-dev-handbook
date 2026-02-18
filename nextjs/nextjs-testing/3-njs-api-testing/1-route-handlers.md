---
source_course: "nextjs-testing"
source_lesson: "nextjs-testing-njs-testing-route-handlers"
---

# Testing Route Handlers

Route Handlers export standard functions that can be called directly in tests.

## Example Route Handler

```typescript
// app/api/users/route.ts
import { prisma } from '@/lib/prisma';

export async function GET() {
  const users = await prisma.user.findMany();
  return Response.json(users);
}

export async function POST(request: Request) {
  const data = await request.json();
  const user = await prisma.user.create({ data });
  return Response.json(user, { status: 201 });
}
```

## Testing

```typescript
// app/api/users/route.test.ts
import { GET, POST } from './route';

jest.mock('@/lib/prisma', () => ({
  prisma: {
    user: {
      findMany: jest.fn().mockResolvedValue([{ id: 1, name: 'John' }]),
      create: jest.fn().mockImplementation((args) => ({ id: 2, ...args.data })),
    },
  },
}));

describe('Users API', () => {
  it('GET returns users', async () => {
    const response = await GET();
    const data = await response.json();

    expect(response.status).toBe(200);
    expect(data).toHaveLength(1);
    expect(data[0].name).toBe('John');
  });

  it('POST creates a user', async () => {
    const request = new Request('http://localhost/api/users', {
      method: 'POST',
      body: JSON.stringify({ name: 'Jane', email: 'jane@example.com' }),
    });

    const response = await POST(request);

    expect(response.status).toBe(201);
  });
});
```

---

> 📘 *This lesson is part of the [Next.js Testing Strategies](https://stanza.dev/courses/nextjs-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*