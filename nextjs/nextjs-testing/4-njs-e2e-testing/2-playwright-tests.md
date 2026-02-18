---
source_course: "nextjs-testing"
source_lesson: "nextjs-testing-njs-playwright-tests"
---

# Writing E2E Tests

Playwright tests simulate real user interactions.

## Basic Test

```typescript
// e2e/home.spec.ts
import { test, expect } from '@playwright/test';

test('homepage has title', async ({ page }) => {
  await page.goto('/');

  await expect(page).toHaveTitle(/My App/);
});

test('navigation works', async ({ page }) => {
  await page.goto('/');

  await page.click('a[href="/about"]');

  await expect(page).toHaveURL('/about');
  await expect(page.locator('h1')).toContainText('About');
});
```

## Form Interaction

```typescript
test('can create a todo', async ({ page }) => {
  await page.goto('/todos');

  await page.fill('input[name="title"]', 'Buy groceries');
  await page.click('button[type="submit"]');

  await expect(page.locator('.todo-item')).toContainText('Buy groceries');
});
```

## API Mocking

```typescript
test('handles API errors', async ({ page }) => {
  await page.route('/api/todos', (route) => {
    route.fulfill({
      status: 500,
      body: JSON.stringify({ error: 'Server error' }),
    });
  });

  await page.goto('/todos');
  await expect(page.locator('.error')).toBeVisible();
});
```

---

> 📘 *This lesson is part of the [Next.js Testing Strategies](https://stanza.dev/courses/nextjs-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*