---
source_course: "svelte-5-enterprise"
source_lesson: "svelte-5-enterprise-testing-props-slots"
---

# Testing Component APIs

Test how components receive and use props and snippets.

## Testing Props

```javascript
// Alert.svelte
<script>
  let { type = 'info', message, ondismiss } = $props();
</script>

<div class="alert alert-{type}" role="alert">
  {message}
  {#if ondismiss}
    <button onclick={ondismiss}>×</button>
  {/if}
</div>
```

```javascript
// Alert.test.js
test('renders with correct type class', () => {
  const { container } = render(Alert, {
    props: { type: 'error', message: 'Oops!' }
  });
  
  expect(container.querySelector('.alert-error')).toBeInTheDocument();
});

test('shows dismiss button when ondismiss provided', () => {
  const { getByRole } = render(Alert, {
    props: { message: 'Test', ondismiss: vi.fn() }
  });
  
  expect(getByRole('button', { name: '×' })).toBeInTheDocument();
});

test('hides dismiss button when no ondismiss', () => {
  const { queryByRole } = render(Alert, {
    props: { message: 'Test' }
  });
  
  expect(queryByRole('button')).not.toBeInTheDocument();
});
```

## Testing with Snippets (Svelte 5)

For components that accept snippets, you may need to wrap them:

```javascript
// Card.svelte
<script>
  let { title, children, footer } = $props();
</script>

<div class="card">
  <h2>{title}</h2>
  <div class="body">{@render children()}</div>
  {#if footer}
    <div class="footer">{@render footer()}</div>
  {/if}
</div>
```

```javascript
// Card.test.js
import { render } from '@testing-library/svelte';
import Card from './Card.svelte';

// Create a wrapper component for testing
const TestCard = {
  Component: Card,
  props: {
    title: 'Test Title',
    children: () => 'Card content here',
    footer: () => 'Footer content'
  }
};

test('renders children', () => {
  // Note: Testing snippets may require wrapper components
  // or using Vitest browser mode for full support
});
```

## Rerendering with New Props

```javascript
test('updates when props change', async () => {
  const { getByText, rerender } = render(Greeting, {
    props: { name: 'Alice' }
  });
  
  expect(getByText('Hello, Alice!')).toBeInTheDocument();
  
  await rerender({ name: 'Bob' });
  
  expect(getByText('Hello, Bob!')).toBeInTheDocument();
});
```

📖 [Testing Library Svelte](https://testing-library.com/docs/svelte-testing-library/api)

---

> 📘 *This lesson is part of the [Enterprise Svelte: Testing & Architecture](https://stanza.dev/courses/svelte-5-enterprise) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*