---
source_course: "nextjs-testing"
source_lesson: "nextjs-testing-njs-playwright-setup"
---

# End-to-End Testing with Playwright

Playwright allows you to test your entire application in real browsers.

## Installation

```bash
npm init playwright@latest
```

This creates:
- `playwright.config.ts`
- `tests/` directory
- Example tests

## Configuration for Next.js

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
  use: {
    baseURL: 'http://localhost:3000',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit', use: { ...devices['Desktop Safari'] } },
  ],
});
```

The `webServer` option automatically starts your Next.js dev server before tests.

## Resources

- [Playwright with Next.js](https://nextjs.org/docs/app/building-your-application/testing/playwright) — Official Playwright integration guide

---

> 📘 *This lesson is part of the [Next.js Testing Strategies](https://stanza.dev/courses/nextjs-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*