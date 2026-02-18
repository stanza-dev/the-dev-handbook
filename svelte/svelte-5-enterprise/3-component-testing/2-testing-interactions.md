---
source_course: "svelte-5-enterprise"
source_lesson: "svelte-5-enterprise-testing-interactions"
---

# Simulating User Behavior

Test how users interact with your components.

## Clicking

```javascript
import { render, fireEvent } from '@testing-library/svelte';

test('counter increments', async () => {
  const { getByRole, getByText } = render(Counter);
  
  await fireEvent.click(getByRole('button', { name: '+' }));
  
  expect(getByText('Count: 1')).toBeInTheDocument();
});
```

## Typing

```javascript
test('updates input value', async () => {
  const { getByRole } = render(SearchInput);
  const input = getByRole('textbox');
  
  await fireEvent.input(input, { target: { value: 'hello' }});
  
  expect(input).toHaveValue('hello');
});

// Or use userEvent for more realistic typing
import userEvent from '@testing-library/user-event';

test('types character by character', async () => {
  const user = userEvent.setup();
  const { getByRole } = render(SearchInput);
  
  await user.type(getByRole('textbox'), 'hello');
  
  expect(getByRole('textbox')).toHaveValue('hello');
});
```

## Form Submission

```javascript
test('submits form', async () => {
  const onsubmit = vi.fn();
  const { getByRole, getByLabelText } = render(ContactForm, {
    props: { onsubmit }
  });
  
  await fireEvent.input(getByLabelText('Email'), {
    target: { value: 'test@example.com' }
  });
  await fireEvent.input(getByLabelText('Message'), {
    target: { value: 'Hello!' }
  });
  await fireEvent.click(getByRole('button', { name: 'Send' }));
  
  expect(onsubmit).toHaveBeenCalledWith({
    email: 'test@example.com',
    message: 'Hello!'
  });
});
```

## Keyboard Events

```javascript
test('handles escape key', async () => {
  const onclose = vi.fn();
  const { getByRole } = render(Modal, { props: { onclose }});
  
  await fireEvent.keyDown(getByRole('dialog'), { key: 'Escape' });
  
  expect(onclose).toHaveBeenCalled();
});
```

## Waiting for Async Updates

```javascript
import { render, waitFor } from '@testing-library/svelte';

test('loads data', async () => {
  const { getByText, findByText } = render(UserList);
  
  // Initially shows loading
  expect(getByText('Loading...')).toBeInTheDocument();
  
  // Wait for data to load
  expect(await findByText('Alice')).toBeInTheDocument();
});
```

📖 [fireEvent docs](https://testing-library.com/docs/dom-testing-library/api-events)

---

> 📘 *This lesson is part of the [Enterprise Svelte: Testing & Architecture](https://stanza.dev/courses/svelte-5-enterprise) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*