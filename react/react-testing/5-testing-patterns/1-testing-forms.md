---
source_course: "react-testing"
source_lesson: "react-testing-testing-forms"
---

# Testing Forms

Forms are a critical part of most applications. Here's how to test them thoroughly.

## Basic Form Test

```jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

test('submits form with user data', async () => {
  const user = userEvent.setup();
  const onSubmit = jest.fn();
  
  render(<ContactForm onSubmit={onSubmit} />);
  
  await user.type(screen.getByLabelText('Name'), 'John Doe');
  await user.type(screen.getByLabelText('Email'), 'john@example.com');
  await user.type(screen.getByLabelText('Message'), 'Hello!');
  
  await user.click(screen.getByRole('button', { name: 'Submit' }));
  
  expect(onSubmit).toHaveBeenCalledWith({
    name: 'John Doe',
    email: 'john@example.com',
    message: 'Hello!'
  });
});
```

## Testing Validation

```jsx
test('shows validation errors for empty fields', async () => {
  const user = userEvent.setup();
  render(<ContactForm />);
  
  // Submit empty form
  await user.click(screen.getByRole('button', { name: 'Submit' }));
  
  // Check for errors
  expect(screen.getByText('Name is required')).toBeInTheDocument();
  expect(screen.getByText('Email is required')).toBeInTheDocument();
});

test('shows error for invalid email', async () => {
  const user = userEvent.setup();
  render(<ContactForm />);
  
  await user.type(screen.getByLabelText('Email'), 'invalid-email');
  await user.tab(); // Trigger blur validation
  
  expect(screen.getByText('Invalid email address')).toBeInTheDocument();
});

test('clears error when user fixes input', async () => {
  const user = userEvent.setup();
  render(<ContactForm />);
  
  // Trigger error
  await user.click(screen.getByRole('button', { name: 'Submit' }));
  expect(screen.getByText('Email is required')).toBeInTheDocument();
  
  // Fix the error
  await user.type(screen.getByLabelText('Email'), 'valid@email.com');
  
  // Error should be gone
  expect(screen.queryByText('Email is required')).not.toBeInTheDocument();
});
```

## Testing Submit States

```jsx
test('disables submit button while submitting', async () => {
  const user = userEvent.setup();
  
  render(<ContactForm />);
  
  // Fill form
  await user.type(screen.getByLabelText('Name'), 'John');
  await user.type(screen.getByLabelText('Email'), 'john@test.com');
  
  const submitButton = screen.getByRole('button', { name: 'Submit' });
  await user.click(submitButton);
  
  // Button should be disabled during submission
  expect(submitButton).toBeDisabled();
  expect(submitButton).toHaveTextContent('Submitting...');
  
  // Wait for completion
  await waitFor(() => {
    expect(submitButton).toBeEnabled();
  });
});
```

## Testing Select Inputs

```jsx
test('selects country from dropdown', async () => {
  const user = userEvent.setup();
  render(<AddressForm />);
  
  await user.selectOptions(
    screen.getByLabelText('Country'),
    'USA'
  );
  
  expect(screen.getByLabelText('Country')).toHaveValue('USA');
});
```

---

> 📘 *This lesson is part of the [React Testing Strategies](https://stanza.dev/courses/react-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*