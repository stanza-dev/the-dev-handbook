---
source_course: "nextjs-testing"
source_lesson: "nextjs-testing-njs-visual-testing"
---

# Visual Regression Testing

## Introduction

Visual regression tests capture screenshots and compare them against baselines. They catch CSS bugs that unit tests miss.

## Key Concepts

**Visual testing**:

- Captures full-page or component screenshots
- Compares against baseline images
- Highlights pixel-level differences

## Deep Dive

### Playwright Visual Testing

```typescript
import { test, expect } from '@playwright/test';

test('home page visual', async ({ page }) => {
  await page.goto('/');
  await expect(page).toHaveScreenshot('home.png');
});

test('button states', async ({ page }) => {
  await page.goto('/components');
  
  const button = page.getByRole('button', { name: 'Submit' });
  await expect(button).toHaveScreenshot('button-default.png');
  
  await button.hover();
  await expect(button).toHaveScreenshot('button-hover.png');
});
```

### Updating Baselines

```bash
npx playwright test --update-snapshots
```

### Handling Dynamic Content

```typescript
test('dashboard', async ({ page }) => {
  await page.goto('/dashboard');
  
  // Mask dynamic content
  await expect(page).toHaveScreenshot('dashboard.png', {
    mask: [
      page.locator('.timestamp'),
      page.locator('.user-avatar'),
    ],
  });
});
```

## Summary

Visual regression testing catches CSS and layout bugs that other tests miss. Use Playwright's toHaveScreenshot() for simple setups, mask dynamic content, and review diffs carefully before updating baselines.

## Resources

- [Visual Comparisons](https://playwright.dev/docs/test-snapshots) — Playwright visual testing documentation

---

> 📘 *This lesson is part of the [Next.js Testing Strategies](https://stanza.dev/courses/nextjs-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*