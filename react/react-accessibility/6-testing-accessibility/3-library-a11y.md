---
source_course: "react-accessibility"
source_lesson: "react-accessibility-react-testing-library-a11y"
---

# Accessibility in React Testing Library

RTL encourages accessible queries by default.

## Accessible Queries

RTL query priority encourages accessibility:

```jsx
// ✅ Best: Accessible queries
screen.getByRole('button', { name: 'Submit' });
screen.getByLabelText('Email');
screen.getByText('Welcome');

// ⚠️ Less preferred
screen.getByTestId('submit-button');
```

## Testing by Role

```jsx
test('button is accessible', () => {
  render(<Button>Click me</Button>);
  
  const button = screen.getByRole('button', { name: 'Click me' });
  expect(button).toBeInTheDocument();
});

test('form inputs have labels', () => {
  render(<LoginForm />);
  
  expect(screen.getByLabelText('Email')).toBeInTheDocument();
  expect(screen.getByLabelText('Password')).toBeInTheDocument();
});
```

## Testing ARIA States

```jsx
test('checkbox can be toggled', async () => {
  const user = userEvent.setup();
  render(<Checkbox label="Accept terms" />);
  
  const checkbox = screen.getByRole('checkbox', { name: 'Accept terms' });
  expect(checkbox).not.toBeChecked();
  
  await user.click(checkbox);
  expect(checkbox).toBeChecked();
});

test('expanded state updates', async () => {
  const user = userEvent.setup();
  render(<Accordion title="Details" />);
  
  const button = screen.getByRole('button', { name: 'Details' });
  expect(button).toHaveAttribute('aria-expanded', 'false');
  
  await user.click(button);
  expect(button).toHaveAttribute('aria-expanded', 'true');
});
```

## Testing Focus Management

```jsx
test('modal focuses on open', async () => {
  const user = userEvent.setup();
  render(<ModalTrigger />);
  
  await user.click(screen.getByRole('button', { name: 'Open modal' }));
  
  const dialog = screen.getByRole('dialog');
  expect(dialog).toHaveFocus();
});

test('focus returns to trigger on close', async () => {
  const user = userEvent.setup();
  render(<ModalTrigger />);
  
  const openButton = screen.getByRole('button', { name: 'Open modal' });
  await user.click(openButton);
  
  await user.click(screen.getByRole('button', { name: 'Close' }));
  expect(openButton).toHaveFocus();
});
```

## Testing Keyboard Navigation

```jsx
test('tabs navigate with arrow keys', async () => {
  const user = userEvent.setup();
  render(<Tabs />);
  
  const firstTab = screen.getByRole('tab', { name: 'Tab 1' });
  firstTab.focus();
  
  await user.keyboard('{ArrowRight}');
  
  expect(screen.getByRole('tab', { name: 'Tab 2' })).toHaveFocus();
});
```

## Combining with jest-axe

```jsx
test('component has no a11y violations', async () => {
  const { container } = render(<ComplexComponent />);
  
  // Functional tests
  expect(screen.getByRole('heading')).toHaveTextContent('Title');
  
  // Accessibility tests
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});
```

---

> 📘 *This lesson is part of the [React Accessibility (a11y)](https://stanza.dev/courses/react-accessibility) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*