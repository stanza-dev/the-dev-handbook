---
source_course: "nextjs-testing"
source_lesson: "nextjs-testing-njs-msw"
---

# Mock Service Worker (MSW)

MSW intercepts network requests at the network level, making it perfect for testing.

## Installation

```bash
npm install -D msw
```

## Setup Handlers

```typescript
// mocks/handlers.ts
import { http, HttpResponse } from 'msw';

export const handlers = [
  http.get('/api/users', () => {
    return HttpResponse.json([
      { id: 1, name: 'John' },
      { id: 2, name: 'Jane' },
    ]);
  }),

  http.post('/api/users', async ({ request }) => {
    const data = await request.json();
    return HttpResponse.json({ id: 3, ...data }, { status: 201 });
  }),
];
```

## Setup Server for Tests

```typescript
// mocks/server.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);
```

```typescript
// jest.setup.js
import { server } from './mocks/server';

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

Now all fetch requests in your tests will be intercepted!

---

> 📘 *This lesson is part of the [Next.js Testing Strategies](https://stanza.dev/courses/nextjs-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*