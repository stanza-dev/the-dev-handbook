---
source_course: "svelte-5-enterprise"
source_lesson: "svelte-5-enterprise-testing-library-basics"
---

# Component Testing with Testing Library

Svelte Testing Library follows the principle: test behavior, not implementation.

## Basic Component Test

```javascript
// Button.svelte
<script>
  let { label, onclick } = $props();
</script>

<button {onclick}>{label}</button>
```

```javascript
// Button.test.js
import { render, fireEvent } from '@testing-library/svelte';
import Button from './Button.svelte';

test('renders label', () => {
  const { getByRole } = render(Button, { props: { label: 'Click me' }});
  
  expect(getByRole('button')).toHaveTextContent('Click me');
});

test('calls onclick when clicked', async () => {
  const onclick = vi.fn();
  const { getByRole } = render(Button, { 
    props: { label: 'Click', onclick } 
  });
  
  await fireEvent.click(getByRole('button'));
  
  expect(onclick).toHaveBeenCalled();
});
```

## Query Methods

| Method | Returns | Throws if not found |
|--------|---------|--------------------|
| `getBy...` | Element | Yes |
| `queryBy...` | Element or null | No |
| `findBy...` | Promise<Element> | Yes |
| `getAllBy...` | Element[] | Yes |

## Query Types

```javascript
const { 
  getByRole,       // Accessibility role (button, textbox, etc.)
  getByText,       // Text content
  getByLabelText,  // Form label
  getByPlaceholderText,
  getByTestId,     // data-testid attribute (last resort)
} = render(Component);
```

## Priority Order

1. **Accessible queries** (getByRole, getByLabelText) - What assistive tech sees
2. **Semantic queries** (getByText, getByTitle) - What users see
3. **Test IDs** (getByTestId) - Last resort

📖 [Testing Library queries](https://testing-library.com/docs/queries/about)

---

> 📘 *This lesson is part of the [Enterprise Svelte: Testing & Architecture](https://stanza.dev/courses/svelte-5-enterprise) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*