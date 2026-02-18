---
source_course: "react-testing"
source_lesson: "react-testing-testing-modals"
---

# Testing Modals & Portals

Modals often use portals, which render outside the component tree. Here's how to test them.

## Basic Modal Test

```jsx
test('opens modal when button clicked', async () => {
  const user = userEvent.setup();
  render(<ModalExample />);
  
  // Modal not visible initially
  expect(screen.queryByRole('dialog')).not.toBeInTheDocument();
  
  // Open modal
  await user.click(screen.getByRole('button', { name: 'Open Modal' }));
  
  // Modal is now visible
  expect(screen.getByRole('dialog')).toBeInTheDocument();
  expect(screen.getByText('Modal Title')).toBeInTheDocument();
});

test('closes modal on close button', async () => {
  const user = userEvent.setup();
  render(<ModalExample />);
  
  // Open modal
  await user.click(screen.getByRole('button', { name: 'Open Modal' }));
  
  // Close modal
  await user.click(screen.getByRole('button', { name: 'Close' }));
  
  // Modal should be gone
  await waitForElementToBeRemoved(
    screen.queryByRole('dialog')
  );
});
```

## Testing Backdrop Click

```jsx
test('closes modal when clicking backdrop', async () => {
  const user = userEvent.setup();
  render(<Modal isOpen onClose={jest.fn()} />);
  
  const backdrop = screen.getByTestId('modal-backdrop');
  await user.click(backdrop);
  
  expect(onClose).toHaveBeenCalled();
});
```

## Testing Escape Key

```jsx
test('closes modal on Escape key', async () => {
  const user = userEvent.setup();
  const onClose = jest.fn();
  
  render(<Modal isOpen onClose={onClose}><p>Content</p></Modal>);
  
  await user.keyboard('{Escape}');
  
  expect(onClose).toHaveBeenCalled();
});
```

## Testing Focus Trap

```jsx
test('traps focus within modal', async () => {
  const user = userEvent.setup();
  render(
    <Modal isOpen>
      <input data-testid="input-1" />
      <input data-testid="input-2" />
      <button>Close</button>
    </Modal>
  );
  
  // First element should be focused
  expect(screen.getByTestId('input-1')).toHaveFocus();
  
  // Tab through elements
  await user.tab();
  expect(screen.getByTestId('input-2')).toHaveFocus();
  
  await user.tab();
  expect(screen.getByRole('button', { name: 'Close' })).toHaveFocus();
  
  // Tab should cycle back to first element
  await user.tab();
  expect(screen.getByTestId('input-1')).toHaveFocus();
});
```

## Portal Considerations

```jsx
// Portals render to document.body by default
// RTL queries still find them!

test('finds portal content', async () => {
  render(<ModalWithPortal isOpen />);
  
  // This works even though content is in a portal
  expect(screen.getByRole('dialog')).toBeInTheDocument();
});
```

---

> 📘 *This lesson is part of the [React Testing Strategies](https://stanza.dev/courses/react-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*