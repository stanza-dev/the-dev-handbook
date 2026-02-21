---
source_course: "react-intermediate"
source_lesson: "react-testing-patterns"
---

# Testing Patterns and Strategies

## Introduction

Beyond basic rendering and clicking, real-world components need testing strategies for mocking API calls, handling context providers, testing custom hooks, and structuring tests for maintainability. This lesson covers the patterns that make React tests robust and practical.

## Key Concepts

- **Custom render function**: A wrapper around RTL's render that includes providers (QueryClient, Auth, Theme) so every test gets the necessary context.
- **MSW (Mock Service Worker)**: A library that intercepts network requests at the service worker level, providing realistic API mocking without modifying component code.
- **renderHook**: A utility for testing custom hooks in isolation without creating a test component.

## Real World Context

A component uses TanStack Query, reads from AuthContext, and applies theme from ThemeContext. Writing boilerplate to wrap each test in three providers is tedious and error-prone. A custom render function handles this once, making every test concise and consistent.

## Deep Dive

**Custom render with providers:**

```tsx
import { render, RenderOptions } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { AuthProvider } from '@/features/auth';

function createTestQueryClient() {
  return new QueryClient({
    defaultOptions: { queries: { retry: false } },
  });
}

function AllProviders({ children }: { children: React.ReactNode }) {
  const queryClient = createTestQueryClient();
  return (
    <QueryClientProvider client={queryClient}>
      <AuthProvider>
        {children}
      </AuthProvider>
    </QueryClientProvider>
  );
}

function customRender(ui: React.ReactElement, options?: RenderOptions) {
  return render(ui, { wrapper: AllProviders, ...options });
}

export { customRender as render, screen } from '@testing-library/react';
```

**Mocking API calls with MSW:**

```tsx
import { http, HttpResponse } from 'msw';
import { setupServer } from 'msw/node';

const server = setupServer(
  http.get('/api/users/:id', ({ params }) => {
    return HttpResponse.json({ id: params.id, name: 'Test User' });
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

**Testing custom hooks:**

```tsx
import { renderHook, waitFor } from '@testing-library/react';

it('returns user data', async () => {
  const { result } = renderHook(() => useUser('123'), { wrapper: AllProviders });
  await waitFor(() => expect(result.current.data).toBeDefined());
  expect(result.current.data.name).toBe('Test User');
});
```

## Common Pitfalls

1. **Mocking fetch globally** — Mocking fetch with jest.fn() is fragile and does not test the actual request format. MSW provides more realistic and maintainable API mocking.
2. **Not creating a custom render** — Manually wrapping components in providers for every test leads to duplication and inconsistent test setups.

## Best Practices

1. **Create a custom render with all providers** — Set it up once in a test utility file and import it instead of RTL's render.
2. **Use MSW for API mocking** — It intercepts at the network level, so your components use their real fetch code.

## Summary

- Custom render functions eliminate provider boilerplate in tests.
- MSW provides realistic API mocking at the network level.
- renderHook allows testing custom hooks in isolation.

## Code Examples

**Custom render function with providers for testing**

```tsx
// test-utils.tsx
import { render, RenderOptions } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

function AllProviders({ children }: { children: React.ReactNode }) {
  const client = new QueryClient({ defaultOptions: { queries: { retry: false } } });
  return <QueryClientProvider client={client}>{children}</QueryClientProvider>;
}

const customRender = (ui: React.ReactElement, opts?: RenderOptions) =>
  render(ui, { wrapper: AllProviders, ...opts });

export { customRender as render };
```


## Resources

- [Testing Library Setup](https://testing-library.com/docs/react-testing-library/setup) — Official setup guide for React Testing Library

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*