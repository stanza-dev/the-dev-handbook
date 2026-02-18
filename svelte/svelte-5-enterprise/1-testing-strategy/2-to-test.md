---
source_course: "svelte-5-enterprise"
source_lesson: "svelte-5-enterprise-what-to-test"
---

# Testing the Right Things

Not everything needs a test. Focus on what provides value.

## ✅ DO Test

**Business Logic**
```javascript
// ✅ Test: Complex calculations
test('applies discount correctly', () => {
  const order = new Order({ subtotal: 100 });
  order.applyDiscount('SAVE20');
  expect(order.total).toBe(80);
});
```

**User Interactions**
```javascript
// ✅ Test: Form submission works
test('submits form with data', async () => {
  const { getByLabelText, getByRole } = render(ContactForm);
  await fireEvent.input(getByLabelText('Email'), { target: { value: 'test@example.com' }});
  await fireEvent.click(getByRole('button', { name: 'Submit' }));
  // Assert form was submitted
});
```

**Edge Cases**
```javascript
// ✅ Test: Empty state, errors, boundaries
test('shows empty state when no items', () => {
  const { getByText } = render(ItemList, { props: { items: [] }});
  expect(getByText('No items yet')).toBeInTheDocument();
});
```

**Accessibility**
```javascript
// ✅ Test: Keyboard navigation works
test('can navigate with keyboard', async () => {
  const { getByRole } = render(Dropdown);
  await fireEvent.keyDown(getByRole('button'), { key: 'ArrowDown' });
  expect(getByRole('option', { name: 'First' })).toHaveFocus();
});
```

## ❌ DON'T Test

**Implementation Details**
```javascript
// ❌ Don't test internal state names
test('sets isLoading to true', () => {
  // Testing internal state, not behavior
});

// ✅ Test behavior instead
test('shows loading spinner during fetch', async () => {
  const { getByRole } = render(DataLoader);
  expect(getByRole('progressbar')).toBeInTheDocument();
});
```

**Framework Features**
```javascript
// ❌ Don't test that Svelte's reactivity works
test('$state updates component', () => {
  // Svelte already tests this!
});
```

**Third-Party Code**
```javascript
// ❌ Don't test libraries you don't own
test('fetch returns JSON', () => {
  // Test YOUR code that uses fetch
});
```

📖 [What to test](https://kentcdodds.com/blog/write-tests)

---

> 📘 *This lesson is part of the [Enterprise Svelte: Testing & Architecture](https://stanza.dev/courses/svelte-5-enterprise) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*