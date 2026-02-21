---
source_course: "react-intermediate"
source_lesson: "react-testing-async"
---

# Testing Asynchronous Components

## Introduction

Most real-world components involve asynchronous operations: fetching data, waiting for user input debouncing, or responding to timers. Testing these requires special techniques to wait for updates and assert on the final state.

## Key Concepts

- **findBy queries**: Async queries that wait for an element to appear. They retry until the element is found or a timeout is reached.
- **waitFor**: A utility that retries an assertion until it passes, useful for waiting on state updates triggered by async operations.
- **act**: Ensures all state updates and effects are flushed before assertions. RTL wraps most operations in act automatically, but manual wrapping is sometimes needed.

## Real World Context

A search component debounces input by 300ms before fetching results. The test types a query, but the results do not appear immediately because of the debounce delay and async fetch. Using findBy and waitFor, the test waits for the results to appear without using arbitrary setTimeout delays.

## Deep Dive

**Using findBy for async elements:**

```tsx
it('displays user data after loading', async () => {
  render(<UserProfile userId="123" />);

  // Check loading state is shown initially
  expect(screen.getByText(/loading/i)).toBeInTheDocument();

  // Wait for user data to appear
  const heading = await screen.findByText('John Doe');
  expect(heading).toBeInTheDocument();

  // Verify loading is gone
  expect(screen.queryByText(/loading/i)).not.toBeInTheDocument();
});
```

**Using waitFor for complex assertions:**

```tsx
it('updates count after async operation', async () => {
  render(<AsyncCounter />);
  await userEvent.click(screen.getByRole('button', { name: /increment/i }));

  // waitFor retries until the assertion passes
  await waitFor(() => {
    expect(screen.getByText(/count: 1/i)).toBeInTheDocument();
  });
});
```

**Testing loading and error states:**

```tsx
it('shows error when fetch fails', async () => {
  // Override MSW handler for this test
  server.use(
    http.get('/api/users/:id', () => {
      return new HttpResponse(null, { status: 500 });
    })
  );

  render(<UserProfile userId="123" />);
  const errorMessage = await screen.findByText(/failed to load/i);
  expect(errorMessage).toBeInTheDocument();
});
```

**Fake timers for debounce testing:**

```tsx
it('debounces search input', async () => {
  jest.useFakeTimers();
  render(<SearchInput onSearch={mockSearch} />);
  await userEvent.type(screen.getByRole('searchbox'), 'react');
  jest.advanceTimersByTime(300);
  expect(mockSearch).toHaveBeenCalledWith('react');
  jest.useRealTimers();
});
```

## Common Pitfalls

1. **Using setTimeout in tests** — Arbitrary delays make tests slow and flaky. Use findBy, waitFor, or fake timers instead.
2. **Not waiting for async updates** — Asserting immediately after triggering an async operation fails because the DOM has not updated yet. Always use findBy or waitFor.

## Best Practices

1. **Use findBy for elements that appear asynchronously** — It handles the waiting and retry logic for you.
2. **Use fake timers for debounce and timer testing** — Control time precisely instead of waiting real milliseconds.

## Summary

- Use findBy queries for elements that appear after async operations.
- Use waitFor for complex assertions that need retry logic.
- Use fake timers to test debounced inputs and timed behavior without real delays.

## Code Examples

**Testing async data loading with findBy and MSW**

```tsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

it('loads and displays user data', async () => {
  render(<UserProfile userId="123" />);

  // Wait for async data to appear
  expect(await screen.findByText('John Doe')).toBeInTheDocument();

  // Verify loading state is gone
  expect(screen.queryByText(/loading/i)).not.toBeInTheDocument();
});

it('shows error on failure', async () => {
  server.use(
    http.get('/api/users/:id', () => new HttpResponse(null, { status: 500 }))
  );
  render(<UserProfile userId="123" />);
  expect(await screen.findByText(/failed/i)).toBeInTheDocument();
});
```


## Resources

- [Async Methods](https://testing-library.com/docs/dom-testing-library/api-async) — Testing Library async utilities documentation

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*