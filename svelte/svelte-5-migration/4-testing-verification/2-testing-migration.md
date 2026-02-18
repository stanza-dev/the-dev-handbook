---
source_course: "svelte-5-migration"
source_lesson: "svelte-5-migration-testing-migration"
---

# Verifying Your Migration

Thorough testing is crucial during migration. Here's how to catch issues.

## Unit Testing Components

```javascript
// Component.test.js
import { render, fireEvent } from '@testing-library/svelte';
import Counter from './Counter.svelte';

test('counter increments', async () => {
  const { getByText } = render(Counter);
  
  const button = getByText('Increment');
  await fireEvent.click(button);
  
  expect(getByText('Count: 1')).toBeInTheDocument();
});
```

## Common Issues to Test For

**1. Reactivity**
- Does state update when changed?
- Do derived values recalculate?
- Are effects running?

```javascript
test('derived updates when source changes', async () => {
  const { getByText, getByRole } = render(MyComponent);
  
  await fireEvent.click(getByRole('button'));
  
  // Check that derived value updated
  expect(getByText(/doubled: 2/)).toBeInTheDocument();
});
```

**2. Events**
- Are callback props being called?
- Is data passed correctly?

```javascript
test('calls onsubmit with data', async () => {
  const onsubmit = vi.fn();
  const { getByRole } = render(Form, { props: { onsubmit } });
  
  await fireEvent.click(getByRole('button', { name: 'Submit' }));
  
  expect(onsubmit).toHaveBeenCalledWith({ name: 'test' });
});
```

**3. Slots/Snippets**
```javascript
test('renders children', () => {
  const { getByText } = render(Card, {
    props: {
      children: createRawSnippet(() => ({
        render: () => '<p>Content</p>'
      }))
    }
  });
  
  expect(getByText('Content')).toBeInTheDocument();
});
```

## E2E Testing

Run your Playwright/Cypress tests to catch integration issues:

```bash
npm run test:e2e
```

📖 [Testing documentation](https://testing-library.com/docs/svelte-testing-library/intro)

---

> 📘 *This lesson is part of the [Moving to Svelte 5](https://stanza.dev/courses/svelte-5-migration) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*