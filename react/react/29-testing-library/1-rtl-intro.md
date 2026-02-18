---
source_course: "react"
source_lesson: "react-rtl-intro"
---

# React Testing Library

A library for testing React components that encourages good testing practices.

## Philosophy

> The more your tests resemble the way your software is used, the more confidence they can give you.

## Key Principles

1. **Query by accessibility**: Use roles, labels, and text
2. **Avoid implementation details**: Don't test internal state
3. **User-centric testing**: Test what users see and do

## Common Queries

- `getByRole` - Best for accessibility
- `getByText` - For text content
- `getByLabelText` - For form inputs
- `getByTestId` - Last resort

## Code Examples

**React Testing Library examples**

```tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Counter } from './Counter';
import { LoginForm } from './LoginForm';

// Simple component test
describe('Counter', () => {
  it('increments count when button is clicked', async () => {
    render(<Counter />);
    
    // Query by role (accessibility-first)
    const button = screen.getByRole('button', { name: /increment/i });
    const count = screen.getByText(/count: 0/i);
    
    await userEvent.click(button);
    
    expect(screen.getByText(/count: 1/i)).toBeInTheDocument();
  });
});

// Form testing
describe('LoginForm', () => {
  it('submits form with user input', async () => {
    const handleSubmit = jest.fn();
    render(<LoginForm onSubmit={handleSubmit} />);
    
    // Query by label (form inputs)
    const emailInput = screen.getByLabelText(/email/i);
    const passwordInput = screen.getByLabelText(/password/i);
    const submitButton = screen.getByRole('button', { name: /sign in/i });
    
    // Type using userEvent (more realistic)
    await userEvent.type(emailInput, 'test@example.com');
    await userEvent.type(passwordInput, 'password123');
    await userEvent.click(submitButton);
    
    expect(handleSubmit).toHaveBeenCalledWith({
      email: 'test@example.com',
      password: 'password123'
    });
  });
  
  it('shows error for invalid email', async () => {
    render(<LoginForm onSubmit={jest.fn()} />);
    
    await userEvent.type(screen.getByLabelText(/email/i), 'invalid');
    await userEvent.click(screen.getByRole('button', { name: /sign in/i }));
    
    expect(screen.getByText(/invalid email/i)).toBeInTheDocument();
  });
});

// Async testing
describe('UserProfile', () => {
  it('loads and displays user data', async () => {
    render(<UserProfile userId="123" />);
    
    // Check loading state
    expect(screen.getByText(/loading/i)).toBeInTheDocument();
    
    // Wait for data to load
    await waitFor(() => {
      expect(screen.getByText(/john doe/i)).toBeInTheDocument();
    });
  });
});
```


## Resources

- [Testing Library Documentation](https://testing-library.com/docs/react-testing-library/intro/) — Official React Testing Library docs
- [Testing Library Cheatsheet](https://testing-library.com/docs/queries/about/#priority) — Query priority guidelines

---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*