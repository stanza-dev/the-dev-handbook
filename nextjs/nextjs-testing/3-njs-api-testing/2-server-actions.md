---
source_course: "nextjs-testing"
source_lesson: "nextjs-testing-njs-testing-server-actions"
---

# Testing Server Actions

Server Actions are async functions that can be imported and tested directly.

## Example Server Action

```typescript
// actions/todos.ts
'use server';

import { revalidatePath } from 'next/cache';
import { prisma } from '@/lib/prisma';

export async function createTodo(formData: FormData) {
  const title = formData.get('title') as string;

  if (!title) {
    return { error: 'Title is required' };
  }

  await prisma.todo.create({ data: { title } });
  revalidatePath('/todos');

  return { success: true };
}
```

## Testing

```typescript
// actions/todos.test.ts
import { createTodo } from './todos';

jest.mock('@/lib/prisma', () => ({
  prisma: {
    todo: {
      create: jest.fn(),
    },
  },
}));

jest.mock('next/cache', () => ({
  revalidatePath: jest.fn(),
}));

describe('createTodo', () => {
  it('creates a todo with valid data', async () => {
    const formData = new FormData();
    formData.set('title', 'New Todo');

    const result = await createTodo(formData);

    expect(result).toEqual({ success: true });
  });

  it('returns error for missing title', async () => {
    const formData = new FormData();

    const result = await createTodo(formData);

    expect(result).toEqual({ error: 'Title is required' });
  });
});
```

---

> 📘 *This lesson is part of the [Next.js Testing Strategies](https://stanza.dev/courses/nextjs-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*