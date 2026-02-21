---
source_course: "react-intermediate"
source_lesson: "react-rtl-intro"
---

# Introduction to React Testing Library

## Introduction

React Testing Library (RTL) is a testing utility that encourages testing components the way real users interact with them. Instead of testing implementation details like internal state or component instances, RTL focuses on what users see and do: rendered text, accessible roles, and user interactions.

## Key Concepts

- **User-centric testing**: Testing from the user's perspective by querying elements the way a user would find them (by visible text, roles, labels).
- **Query priority**: RTL defines a priority for queries: `getByRole` > `getByLabelText` > `getByText` > `getByTestId`. Accessibility-first queries are preferred.
- **userEvent**: A companion library that simulates realistic user interactions (typing, clicking, tabbing) more accurately than fireEvent.

## Real World Context

A team tests a login form by checking that `component.state.email` equals the typed value. When they refactor from useState to useReducer, all tests break even though the behavior is identical. With RTL, tests query by label text and assert on visible output, making them resilient to implementation changes.

## Deep Dive

RTL provides three categories of queries:

1. **getBy**: Returns the element or throws if not found. Use for elements that should be present.
2. **queryBy**: Returns the element or null. Use for asserting that elements are NOT present.
3. **findBy**: Returns a promise. Use for elements that appear asynchronously.

```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

describe('LoginForm', () => {
  it('submits with valid credentials', async () => {
    const handleSubmit = jest.fn();
    render(<LoginForm onSubmit={handleSubmit} />);

    // Query by accessible role and label
    await userEvent.type(
      screen.getByLabelText(/email/i),
      'user@example.com'
    );
    await userEvent.type(
      screen.getByLabelText(/password/i),
      'secret123'
    );
    await userEvent.click(
      screen.getByRole('button', { name: /sign in/i })
    );

    expect(handleSubmit).toHaveBeenCalledWith({
      email: 'user@example.com',
      password: 'secret123'
    });
  });

  it('shows validation error for empty email', async () => {
    render(<LoginForm onSubmit={jest.fn()} />);
    await userEvent.click(screen.getByRole('button', { name: /sign in/i }));
    expect(screen.getByText(/email is required/i)).toBeInTheDocument();
  });
});
```

The `userEvent` library simulates realistic interactions: `userEvent.type` fires focus, keyDown, keyPress, input, keyUp, and change events in sequence, just like a real keyboard. This catches bugs that `fireEvent.change` would miss.

## Common Pitfalls

1. **Querying by test IDs first** — Test IDs should be a last resort. Prefer getByRole, getByLabelText, and getByText for accessibility-first testing.
2. **Using fireEvent when userEvent is available** — userEvent simulates real user behavior more accurately. Use it for all user interactions.

## Best Practices

1. **Follow the query priority** — getByRole > getByLabelText > getByText > getByTestId. This also improves accessibility.
2. **Test behavior, not implementation** — Assert on what the user sees, not on internal state or component structure.

## Summary

- RTL encourages user-centric testing through accessibility-first queries.
- Use getBy for present elements, queryBy for absent elements, and findBy for async elements.
- Prefer userEvent over fireEvent for realistic interaction simulation.

## Code Examples

**Testing a Counter component with React Testing Library**

```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

describe('Counter', () => {
  it('increments count on click', async () => {
    render(<Counter />);
    const button = screen.getByRole('button', { name: /increment/i });
    expect(screen.getByText(/count: 0/i)).toBeInTheDocument();
    await userEvent.click(button);
    expect(screen.getByText(/count: 1/i)).toBeInTheDocument();
  });
});
```


## Resources

- [Testing Library Documentation](https://testing-library.com/docs/react-testing-library/intro/) — Official React Testing Library docs
- [Query Priority](https://testing-library.com/docs/queries/about/#priority) — Testing Library query priority guidelines

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*