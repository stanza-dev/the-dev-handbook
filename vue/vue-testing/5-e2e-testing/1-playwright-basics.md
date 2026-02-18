---
source_course: "vue-testing"
source_lesson: "vue-testing-playwright-basics"
---

# E2E Testing with Playwright

Playwright tests your application from the user's perspective, running in real browsers.

## Setup

```bash
npm init playwright@latest
```

## Basic Test

```typescript
// tests/example.spec.ts
import { test, expect } from '@playwright/test'

test('homepage has title', async ({ page }) => {
  await page.goto('http://localhost:5173')
  
  await expect(page).toHaveTitle(/My App/)
})

test('navigation works', async ({ page }) => {
  await page.goto('http://localhost:5173')
  
  await page.click('text=About')
  
  await expect(page).toHaveURL(/.*about/)
  await expect(page.locator('h1')).toContainText('About Us')
})
```

## Locators

```typescript
test('finding elements', async ({ page }) => {
  await page.goto('/')
  
  // By text
  await page.click('text=Submit')
  
  // By CSS selector
  await page.fill('.email-input', 'test@example.com')
  
  // By test ID (recommended)
  await page.click('[data-testid="submit-button"]')
  
  // By role
  await page.getByRole('button', { name: 'Submit' }).click()
  
  // By label
  await page.getByLabel('Email').fill('test@example.com')
  
  // By placeholder
  await page.getByPlaceholder('Enter email').fill('test@example.com')
})
```

## Actions

```typescript
test('user actions', async ({ page }) => {
  await page.goto('/')
  
  // Click
  await page.click('button')
  await page.dblclick('.item')
  
  // Type
  await page.fill('input[name="email"]', 'test@example.com')
  await page.type('input', 'slow typing', { delay: 100 })
  
  // Select
  await page.selectOption('select', 'option2')
  
  // Checkbox
  await page.check('input[type="checkbox"]')
  await page.uncheck('input[type="checkbox"]')
  
  // Keyboard
  await page.press('input', 'Enter')
  await page.keyboard.press('Escape')
  
  // Hover
  await page.hover('.dropdown-trigger')
})
```

## Assertions

```typescript
test('assertions', async ({ page }) => {
  await page.goto('/')
  
  // Page assertions
  await expect(page).toHaveTitle('My App')
  await expect(page).toHaveURL(/dashboard/)
  
  // Element assertions
  const heading = page.locator('h1')
  await expect(heading).toBeVisible()
  await expect(heading).toHaveText('Welcome')
  await expect(heading).toContainText('Welc')
  
  // State assertions
  const button = page.locator('button')
  await expect(button).toBeEnabled()
  await expect(button).toBeDisabled()
  
  const checkbox = page.locator('input[type="checkbox"]')
  await expect(checkbox).toBeChecked()
  
  // Count assertions
  const items = page.locator('.item')
  await expect(items).toHaveCount(5)
})
```

## Waiting

```typescript
test('waiting for elements', async ({ page }) => {
  await page.goto('/')
  
  // Wait for element
  await page.waitForSelector('.loaded-content')
  
  // Wait for navigation
  await Promise.all([
    page.waitForNavigation(),
    page.click('a[href="/dashboard"]')
  ])
  
  // Wait for response
  await Promise.all([
    page.waitForResponse('/api/users'),
    page.click('button')
  ])
  
  // Wait for load state
  await page.waitForLoadState('networkidle')
})
```

## Full User Flow Test

```typescript
test('login flow', async ({ page }) => {
  // Navigate to login
  await page.goto('/login')
  
  // Fill form
  await page.getByLabel('Email').fill('user@example.com')
  await page.getByLabel('Password').fill('password123')
  
  // Submit
  await page.getByRole('button', { name: 'Sign In' }).click()
  
  // Verify redirect to dashboard
  await expect(page).toHaveURL('/dashboard')
  await expect(page.getByText('Welcome back')).toBeVisible()
})

test('shopping cart flow', async ({ page }) => {
  await page.goto('/products')
  
  // Add item to cart
  await page.locator('[data-testid="product-1"]')
    .getByRole('button', { name: 'Add to Cart' })
    .click()
  
  // Verify cart count
  await expect(page.locator('.cart-count')).toHaveText('1')
  
  // Go to cart
  await page.click('[data-testid="cart-icon"]')
  await expect(page).toHaveURL('/cart')
  
  // Verify item in cart
  await expect(page.locator('.cart-item')).toHaveCount(1)
  
  // Checkout
  await page.getByRole('button', { name: 'Checkout' }).click()
  await expect(page).toHaveURL('/checkout')
})
```

## Running Tests

```bash
# Run all tests
npx playwright test

# Run specific file
npx playwright test tests/login.spec.ts

# Run with UI mode
npx playwright test --ui

# Run headed (see browser)
npx playwright test --headed

# Debug mode
npx playwright test --debug

# Generate tests
npx playwright codegen localhost:5173
```

## Resources

- [Playwright Documentation](https://playwright.dev/) — Official Playwright documentation

---

> 📘 *This lesson is part of the [Vue Testing Strategies](https://stanza.dev/courses/vue-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*