---
source_course: "svelte-5-enterprise"
source_lesson: "svelte-5-enterprise-playwright-basics"
---

# End-to-End Testing with Playwright

Playwright tests your app in real browsers, testing complete user flows.

## Basic Test Structure

```javascript
// e2e/home.spec.js
import { test, expect } from '@playwright/test';

test('homepage has correct title', async ({ page }) => {
  await page.goto('/');
  
  await expect(page).toHaveTitle(/My App/);
});

test('can navigate to about page', async ({ page }) => {
  await page.goto('/');
  
  await page.getByRole('link', { name: 'About' }).click();
  
  await expect(page).toHaveURL('/about');
  await expect(page.getByRole('heading', { name: 'About Us' })).toBeVisible();
});
```

## Locators

Playwright's locators are similar to Testing Library queries:

```javascript
// By role (preferred)
page.getByRole('button', { name: 'Submit' });
page.getByRole('textbox', { name: 'Email' });
page.getByRole('link', { name: 'Home' });

// By label
page.getByLabel('Email address');

// By placeholder
page.getByPlaceholder('Enter your email');

// By text
page.getByText('Welcome back!');

// By test ID (last resort)
page.getByTestId('submit-button');
```

## Assertions

```javascript
test('element states', async ({ page }) => {
  await page.goto('/form');
  
  // Visibility
  await expect(page.getByRole('button')).toBeVisible();
  await expect(page.getByRole('alert')).not.toBeVisible();
  
  // Enabled/Disabled
  await expect(page.getByRole('button')).toBeEnabled();
  await expect(page.getByRole('button', { name: 'Submit' })).toBeDisabled();
  
  // Text content
  await expect(page.getByRole('heading')).toHaveText('Sign Up');
  await expect(page.getByRole('heading')).toContainText('Sign');
  
  // Attributes
  await expect(page.getByRole('link')).toHaveAttribute('href', '/about');
  
  // URL
  await expect(page).toHaveURL('/dashboard');
});
```

📖 [Playwright documentation](https://playwright.dev/docs/intro)

---

> 📘 *This lesson is part of the [Enterprise Svelte: Testing & Architecture](https://stanza.dev/courses/svelte-5-enterprise) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*