---
source_course: "react-testing"
source_lesson: "react-testing-user-events"
---

# User Events

`@testing-library/user-event` simulates real user interactions better than `fireEvent`.

## Setup

```jsx
import userEvent from '@testing-library/user-event';

test('user interaction', async () => {
  const user = userEvent.setup();
  render(<MyComponent />);
  
  await user.click(screen.getByRole('button'));
});
```

## Click Events

```jsx
test('button click', async () => {
  const user = userEvent.setup();
  const handleClick = jest.fn();
  render(<Button onClick={handleClick}>Click</Button>);
  
  await user.click(screen.getByRole('button'));
  expect(handleClick).toHaveBeenCalled();
});

// Double click
await user.dblClick(element);

// Right click
await user.pointer({ keys: '[MouseRight]', target: element });
```

## Typing

```jsx
test('typing in input', async () => {
  const user = userEvent.setup();
  render(<Input />);
  
  const input = screen.getByRole('textbox');
  await user.type(input, 'Hello World');
  
  expect(input).toHaveValue('Hello World');
});

// Clear and type
await user.clear(input);
await user.type(input, 'New value');

// Type with special keys
await user.type(input, 'Hello{enter}'); // Press enter
await user.type(input, '{selectall}{backspace}'); // Select all, delete
```

## Keyboard Navigation

```jsx
test('keyboard navigation', async () => {
  const user = userEvent.setup();
  render(<Menu />);
  
  // Tab to focus
  await user.tab();
  expect(screen.getByRole('menuitem', { name: 'Home' })).toHaveFocus();
  
  // Arrow keys
  await user.keyboard('{ArrowDown}');
  expect(screen.getByRole('menuitem', { name: 'About' })).toHaveFocus();
  
  // Enter to select
  await user.keyboard('{Enter}');
});
```

## Selection & Options

```jsx
// Select dropdown
test('select option', async () => {
  const user = userEvent.setup();
  render(<CountrySelect />);
  
  await user.selectOptions(
    screen.getByRole('combobox'),
    'France'
  );
  
  expect(screen.getByRole('option', { name: 'France' }).selected).toBe(true);
});

// Checkbox
test('toggle checkbox', async () => {
  const user = userEvent.setup();
  render(<Checkbox label="Accept terms" />);
  
  const checkbox = screen.getByRole('checkbox');
  expect(checkbox).not.toBeChecked();
  
  await user.click(checkbox);
  expect(checkbox).toBeChecked();
});
```

## Why user-event Over fireEvent?

`user-event` simulates full interaction sequences:
- Focus before typing
- Multiple events for a single action
- Realistic event order
- Catches more bugs

```jsx
// fireEvent.click = single click event
// user.click = mouseDown, mouseUp, click, focus, etc.
```

---

> 📘 *This lesson is part of the [React Testing Strategies](https://stanza.dev/courses/react-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*