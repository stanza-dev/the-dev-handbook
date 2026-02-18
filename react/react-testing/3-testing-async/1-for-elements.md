---
source_course: "react-testing"
source_lesson: "react-testing-waiting-for-elements"
---

# Waiting for Elements

React applications are asynchronous. Elements appear after API calls, state updates, and animations.

## findBy Queries

`findBy` queries are the simplest way to wait:

```jsx
test('shows data after loading', async () => {
  render(<UserProfile userId="123" />);
  
  // Wait up to 1 second (default) for element
  const name = await screen.findByText('John Doe');
  expect(name).toBeInTheDocument();
});

// With custom timeout
const element = await screen.findByText('Hello', {}, { timeout: 3000 });
```

## waitFor

For more complex waiting scenarios:

```jsx
import { waitFor } from '@testing-library/react';

test('hides loading spinner after fetch', async () => {
  render(<DataLoader />);
  
  // Initially shows spinner
  expect(screen.getByRole('status')).toBeInTheDocument();
  
  // Wait for spinner to disappear
  await waitFor(() => {
    expect(screen.queryByRole('status')).not.toBeInTheDocument();
  });
});
```

## waitFor Best Practices

```jsx
// ✅ Good: Single assertion in waitFor
await waitFor(() => {
  expect(screen.getByText('Success')).toBeInTheDocument();
});

// ❌ Bad: Multiple assertions
await waitFor(() => {
  expect(screen.getByText('Success')).toBeInTheDocument();
  expect(screen.getByText('Done')).toBeInTheDocument(); // May not run
});

// ✅ Better: Separate waitFor calls
await waitFor(() => expect(screen.getByText('Success')).toBeInTheDocument());
await waitFor(() => expect(screen.getByText('Done')).toBeInTheDocument());
```

## waitForElementToBeRemoved

```jsx
import { waitForElementToBeRemoved } from '@testing-library/react';

test('removes modal after close', async () => {
  const user = userEvent.setup();
  render(<Modal isOpen={true} />);
  
  const modal = screen.getByRole('dialog');
  await user.click(screen.getByRole('button', { name: 'Close' }));
  
  await waitForElementToBeRemoved(modal);
  // OR
  await waitForElementToBeRemoved(() => screen.queryByRole('dialog'));
});
```

## Common Mistakes

```jsx
// ❌ Don't use act() with RTL queries - they handle it
await act(async () => {
  await screen.findByText('Hello');
});

// ✅ Just use findBy
await screen.findByText('Hello');

// ❌ Don't await getBy
await screen.getByText('Hello'); // getBy is synchronous!

// ✅ Use findBy for async
await screen.findByText('Hello');
```

---

> 📘 *This lesson is part of the [React Testing Strategies](https://stanza.dev/courses/react-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*