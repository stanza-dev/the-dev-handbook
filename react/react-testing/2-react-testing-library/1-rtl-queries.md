---
source_course: "react-testing"
source_lesson: "react-testing-rtl-queries"
---

# Queries: Finding Elements

React Testing Library provides queries to find elements in the DOM. Choosing the right query matters!

## Query Priority

Use queries in this order of preference:

### 1. Accessible Queries (Preferred)

```jsx
// By Role (BEST)
screen.getByRole('button', { name: 'Submit' });
screen.getByRole('heading', { level: 1 });
screen.getByRole('textbox', { name: 'Email' });

// By Label Text
screen.getByLabelText('Email Address');

// By Placeholder
screen.getByPlaceholderText('Enter email...');

// By Text Content
screen.getByText('Welcome back!');
```

### 2. Semantic Queries

```jsx
// By Alt Text (images)
screen.getByAltText('User avatar');

// By Title
screen.getByTitle('Close dialog');

// By Display Value (inputs)
screen.getByDisplayValue('john@example.com');
```

### 3. Test IDs (Last Resort)

```jsx
// Only when other queries don't work
screen.getByTestId('custom-element');

// In JSX:
<div data-testid="custom-element">...</div>
```

## Query Variants

Each query comes in three variants:

| Variant | No Match | 1 Match | 1+ Match | Async |
|---------|----------|---------|----------|-------|
| getBy | Throw | Return | Throw | No |
| queryBy | null | Return | Throw | No |
| findBy | Throw | Return | Throw | Yes |

### getBy - Element Should Exist

```jsx
// Throws if not found
const button = screen.getByRole('button');
```

### queryBy - Element Might Not Exist

```jsx
// Returns null if not found
const error = screen.queryByText('Error message');
expect(error).not.toBeInTheDocument();
```

### findBy - Element Will Appear (Async)

```jsx
// Waits for element to appear
const message = await screen.findByText('Success!');
```

## Multiple Elements

Add `All` to find multiple elements:

```jsx
const buttons = screen.getAllByRole('button');
expect(buttons).toHaveLength(3);

const items = screen.queryAllByRole('listitem');
expect(items).toHaveLength(0); // Empty list

const results = await screen.findAllByTestId(/result-/);
```

## Debugging Queries

```jsx
// Print the current DOM
screen.debug();

// Print specific element
screen.debug(screen.getByRole('form'));

// Log accessible roles
import { logRoles } from '@testing-library/react';
logRoles(container);
```

---

> 📘 *This lesson is part of the [React Testing Strategies](https://stanza.dev/courses/react-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*