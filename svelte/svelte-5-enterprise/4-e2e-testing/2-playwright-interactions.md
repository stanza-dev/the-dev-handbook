---
source_course: "svelte-5-enterprise"
source_lesson: "svelte-5-enterprise-playwright-interactions"
---

# Simulating User Actions in Playwright

## Clicking

```javascript
test('button interactions', async ({ page }) => {
  await page.goto('/app');
  
  // Simple click
  await page.getByRole('button', { name: 'Save' }).click();
  
  // Double click
  await page.getByText('Select me').dblclick();
  
  // Right click
  await page.getByText('Context menu').click({ button: 'right' });
  
  // Click with modifiers
  await page.getByText('Multi-select').click({ modifiers: ['Shift'] });
});
```

## Typing

```javascript
test('form input', async ({ page }) => {
  await page.goto('/login');
  
  // Fill input (clears first)
  await page.getByLabel('Email').fill('user@example.com');
  
  // Type character by character (triggers keydown/keyup)
  await page.getByLabel('Password').type('secret123');
  
  // Press specific keys
  await page.getByLabel('Search').press('Enter');
  
  // Clear input
  await page.getByLabel('Email').clear();
});
```

## Selecting Options

```javascript
test('dropdown selection', async ({ page }) => {
  await page.goto('/settings');
  
  // Select by value
  await page.getByLabel('Country').selectOption('us');
  
  // Select by label
  await page.getByLabel('Country').selectOption({ label: 'United States' });
  
  // Multiple selections
  await page.getByLabel('Tags').selectOption(['svelte', 'typescript']);
});
```

## Checkboxes and Radio Buttons

```javascript
test('checkboxes', async ({ page }) => {
  await page.goto('/preferences');
  
  // Check
  await page.getByLabel('Receive emails').check();
  
  // Uncheck
  await page.getByLabel('Receive emails').uncheck();
  
  // Verify state
  await expect(page.getByLabel('Receive emails')).toBeChecked();
});
```

## Drag and Drop

```javascript
test('drag and drop', async ({ page }) => {
  await page.goto('/kanban');
  
  await page.getByText('Task 1').dragTo(
    page.getByText('Done column')
  );
});
```

## Waiting

```javascript
test('wait for elements', async ({ page }) => {
  await page.goto('/dashboard');
  
  // Wait for network idle
  await page.waitForLoadState('networkidle');
  
  // Wait for specific element
  await page.waitForSelector('.data-loaded');
  
  // Wait for response
  await page.waitForResponse('/api/data');
});
```

📖 [Playwright actions](https://playwright.dev/docs/input)

---

> 📘 *This lesson is part of the [Enterprise Svelte: Testing & Architecture](https://stanza.dev/courses/svelte-5-enterprise) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*