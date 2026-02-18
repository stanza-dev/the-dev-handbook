---
source_course: "nextjs-testing"
source_lesson: "nextjs-testing-njs-vitest-alternative"
---

# Vitest: A Faster Alternative

Vitest is a Vite-powered test runner that's becoming increasingly popular.

## Why Vitest?

- **Faster**: Uses Vite's native ESM support.
- **Compatible**: Jest-compatible API.
- **HMR**: Tests re-run instantly on changes.

## Installation

```bash
npm install -D vitest @vitejs/plugin-react jsdom @testing-library/react @testing-library/jest-dom
```

## Configuration

Create `vitest.config.ts`:

```typescript
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    setupFiles: ['./vitest.setup.ts'],
    globals: true,
  },
});
```

Create `vitest.setup.ts`:

```typescript
import '@testing-library/jest-dom/vitest';
```

## Package.json Script

```json
{
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run"
  }
}
```

Vitest runs in watch mode by default, use `vitest run` for CI.

---

> 📘 *This lesson is part of the [Next.js Testing Strategies](https://stanza.dev/courses/nextjs-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*