---
source_course: "nextjs-testing"
source_lesson: "nextjs-testing-njs-testing-server-components"
---

# Testing Server Components

Server Components are async functions that can be tested more simply.

## Rendering Server Components

Server Components can be rendered directly, but you need to handle the async nature:

```typescript
// components/UserProfile.tsx
export async function UserProfile({ userId }: { userId: string }) {
  const user = await fetchUser(userId);
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

```typescript
// components/UserProfile.test.tsx
import { render, screen } from '@testing-library/react';
import { UserProfile } from './UserProfile';

// Mock the fetch function
jest.mock('./lib/api', () => ({
  fetchUser: jest.fn().mockResolvedValue({
    name: 'John Doe',
    email: 'john@example.com',
  }),
}));

describe('UserProfile', () => {
  it('displays user information', async () => {
    // Await the component since it's async
    const Component = await UserProfile({ userId: '1' });
    render(Component);

    expect(screen.getByText('John Doe')).toBeInTheDocument();
    expect(screen.getByText('john@example.com')).toBeInTheDocument();
  });
});
```

## Key Difference

Unlike Client Components, you **await** the Server Component before rendering.

---

> 📘 *This lesson is part of the [Next.js Testing Strategies](https://stanza.dev/courses/nextjs-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*