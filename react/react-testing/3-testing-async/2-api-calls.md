---
source_course: "react-testing"
source_lesson: "react-testing-mocking-api-calls"
---

# Mocking API Calls

Tests should be isolated and not make real network requests. Let's learn different mocking strategies.

## Mock Service Worker (MSW) - Recommended

MSW intercepts requests at the network level:

```jsx
// mocks/handlers.js
import { rest } from 'msw';

export const handlers = [
  rest.get('/api/users/:id', (req, res, ctx) => {
    return res(
      ctx.json({
        id: req.params.id,
        name: 'John Doe',
        email: 'john@example.com'
      })
    );
  }),
  
  rest.post('/api/users', async (req, res, ctx) => {
    const body = await req.json();
    return res(
      ctx.status(201),
      ctx.json({ id: '123', ...body })
    );
  })
];
```

```jsx
// mocks/server.js
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);
```

```jsx
// jest.setup.js
import { server } from './mocks/server';

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

## Using MSW in Tests

```jsx
import { server } from '../mocks/server';
import { rest } from 'msw';

test('displays user data', async () => {
  render(<UserProfile userId="123" />);
  
  // Uses default handler
  await screen.findByText('John Doe');
});

test('handles error', async () => {
  // Override handler for this test
  server.use(
    rest.get('/api/users/:id', (req, res, ctx) => {
      return res(ctx.status(500));
    })
  );
  
  render(<UserProfile userId="123" />);
  
  await screen.findByText(/error/i);
});
```

## Mocking with Jest

```jsx
// Mock the entire module
jest.mock('./api');
import { fetchUser } from './api';

test('displays user', async () => {
  fetchUser.mockResolvedValue({ name: 'Jane' });
  
  render(<UserProfile />);
  
  await screen.findByText('Jane');
});

test('handles rejection', async () => {
  fetchUser.mockRejectedValue(new Error('Network error'));
  
  render(<UserProfile />);
  
  await screen.findByText(/network error/i);
});
```

## Testing Loading States

```jsx
test('shows loading then data', async () => {
  render(<DataList />);
  
  // Initially loading
  expect(screen.getByText('Loading...')).toBeInTheDocument();
  
  // Then data appears
  await screen.findByText('Item 1');
  
  // Loading gone
  expect(screen.queryByText('Loading...')).not.toBeInTheDocument();
});
```

---

> 📘 *This lesson is part of the [React Testing Strategies](https://stanza.dev/courses/react-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*