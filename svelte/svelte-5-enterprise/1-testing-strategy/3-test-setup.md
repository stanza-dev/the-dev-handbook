---
source_course: "svelte-5-enterprise"
source_lesson: "svelte-5-enterprise-test-setup"
---

# Test Environment Setup

Getting your Svelte 5 project ready for testing.

## Installing Dependencies

```bash
# Vitest for unit and component tests
npm install -D vitest @testing-library/svelte @testing-library/jest-dom jsdom

# Playwright for E2E tests
npm install -D @playwright/test
npx playwright install
```

## Vitest Configuration

```javascript
// vitest.config.js
import { defineConfig } from 'vitest/config';
import { svelte } from '@sveltejs/vite-plugin-svelte';

export default defineConfig({
  plugins: [svelte({ hot: !process.env.VITEST })],
  test: {
    include: ['src/**/*.{test,spec}.{js,ts}'],
    environment: 'jsdom',
    globals: true,
    setupFiles: ['./src/test/setup.js']
  }
});
```

## Test Setup File

```javascript
// src/test/setup.js
import '@testing-library/jest-dom';
import { vi } from 'vitest';

// Mock browser APIs if needed
global.ResizeObserver = vi.fn().mockImplementation(() => ({
  observe: vi.fn(),
  unobserve: vi.fn(),
  disconnect: vi.fn()
}));
```

## Package.json Scripts

```json
{
  "scripts": {
    "test": "vitest",
    "test:unit": "vitest run",
    "test:coverage": "vitest run --coverage",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui"
  }
}
```

## Playwright Configuration

```javascript
// playwright.config.js
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  webServer: {
    command: 'npm run dev',
    port: 5173,
    reuseExistingServer: !process.env.CI
  },
  use: {
    baseURL: 'http://localhost:5173'
  }
});
```

📖 [Vitest setup](https://vitest.dev/guide/)

---

> 📘 *This lesson is part of the [Enterprise Svelte: Testing & Architecture](https://stanza.dev/courses/svelte-5-enterprise) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*