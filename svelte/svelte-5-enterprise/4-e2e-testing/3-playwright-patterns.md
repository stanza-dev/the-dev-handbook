---
source_course: "svelte-5-enterprise"
source_lesson: "svelte-5-enterprise-playwright-patterns"
---

# Common E2E Testing Patterns

## Page Object Model

Organize tests with page objects:

```javascript
// pages/LoginPage.js
export class LoginPage {
  constructor(page) {
    this.page = page;
    this.emailInput = page.getByLabel('Email');
    this.passwordInput = page.getByLabel('Password');
    this.submitButton = page.getByRole('button', { name: 'Log in' });
  }
  
  async goto() {
    await this.page.goto('/login');
  }
  
  async login(email, password) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }
}
```

```javascript
// e2e/login.spec.js
import { test, expect } from '@playwright/test';
import { LoginPage } from './pages/LoginPage';

test('successful login', async ({ page }) => {
  const loginPage = new LoginPage(page);
  
  await loginPage.goto();
  await loginPage.login('user@example.com', 'password123');
  
  await expect(page).toHaveURL('/dashboard');
});
```

## Test Fixtures

```javascript
// e2e/fixtures.js
import { test as base } from '@playwright/test';
import { LoginPage } from './pages/LoginPage';
import { DashboardPage } from './pages/DashboardPage';

export const test = base.extend({
  loginPage: async ({ page }, use) => {
    await use(new LoginPage(page));
  },
  dashboardPage: async ({ page }, use) => {
    await use(new DashboardPage(page));
  },
  // Logged in user fixture
  loggedInPage: async ({ page }, use) => {
    await page.goto('/login');
    await page.getByLabel('Email').fill('test@example.com');
    await page.getByLabel('Password').fill('password');
    await page.getByRole('button', { name: 'Log in' }).click();
    await page.waitForURL('/dashboard');
    await use(page);
  }
});
```

```javascript
// e2e/dashboard.spec.js
import { test } from './fixtures';

test('shows user data', async ({ loggedInPage }) => {
  // Already logged in!
  await expect(loggedInPage.getByText('Welcome')).toBeVisible();
});
```

## Visual Regression Testing

```javascript
test('homepage visual', async ({ page }) => {
  await page.goto('/');
  
  await expect(page).toHaveScreenshot('homepage.png');
});

test('component screenshot', async ({ page }) => {
  await page.goto('/components/button');
  
  const button = page.getByRole('button', { name: 'Primary' });
  await expect(button).toHaveScreenshot('primary-button.png');
});
```

📖 [Playwright best practices](https://playwright.dev/docs/best-practices)

---

> 📘 *This lesson is part of the [Enterprise Svelte: Testing & Architecture](https://stanza.dev/courses/svelte-5-enterprise) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*