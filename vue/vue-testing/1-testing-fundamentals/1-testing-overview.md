---
source_course: "vue-testing"
source_lesson: "vue-testing-testing-overview"
---

# Vue Testing Overview

Testing ensures your Vue applications work correctly and remain maintainable as they grow.

## Types of Tests

| Type | Scope | Speed | Confidence |
|------|-------|-------|------------|
| Unit | Single function/composable | Fast | Low-Medium |
| Component | Single component | Medium | Medium |
| Integration | Multiple components | Medium | Medium-High |
| E2E | Entire application | Slow | High |

## Testing Pyramid

```
        /\           E2E (few)
       /  \          - User journeys
      /----\         - Critical paths
     / Intg \        Integration (some)
    /--------\       - Component interactions
   /  Comp   \       Component (many)
  /------------\     - Individual components
 /    Unit      \    Unit (many)
/________________\   - Pure functions
```

## Recommended Tools

### Unit & Component Testing

- **Vitest**: Fast, Vite-native test runner
- **Vue Test Utils**: Official testing utilities for Vue
- **@testing-library/vue**: Testing Library for Vue

### E2E Testing

- **Playwright**: Modern E2E testing
- **Cypress**: Feature-rich E2E framework

## Setup with Vitest

```bash
npm install -D vitest @vue/test-utils happy-dom
```

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  test: {
    environment: 'happy-dom',
    globals: true,
    include: ['**/*.{test,spec}.{js,ts}'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html']
    }
  }
})
```

### Setup File

```typescript
// tests/setup.ts
import { config } from '@vue/test-utils'

// Global plugins for all tests
config.global.plugins = []

// Global stubs
config.global.stubs = {
  // Stub router-link globally
  'router-link': true
}
```

```typescript
// vitest.config.ts
export default defineConfig({
  test: {
    setupFiles: ['./tests/setup.ts']
  }
})
```

## First Test

```typescript
// sum.ts
export function sum(a: number, b: number): number {
  return a + b
}

// sum.test.ts
import { describe, it, expect } from 'vitest'
import { sum } from './sum'

describe('sum', () => {
  it('adds two numbers', () => {
    expect(sum(1, 2)).toBe(3)
  })
  
  it('handles negative numbers', () => {
    expect(sum(-1, 1)).toBe(0)
  })
})
```

## Running Tests

```bash
# Run all tests
npx vitest

# Run in watch mode
npx vitest --watch

# Run specific file
npx vitest sum.test.ts

# Run with coverage
npx vitest --coverage

# Run once (CI)
npx vitest run
```

## Package.json Scripts

```json
{
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run",
    "test:coverage": "vitest run --coverage",
    "test:ui": "vitest --ui"
  }
}
```

## Test Structure

```typescript
import { describe, it, expect, beforeEach, afterEach, vi } from 'vitest'

describe('Feature', () => {
  // Setup before each test
  beforeEach(() => {
    // Reset state
  })
  
  // Cleanup after each test
  afterEach(() => {
    vi.clearAllMocks()
  })
  
  describe('sub-feature', () => {
    it('should do something', () => {
      // Arrange
      const input = 'test'
      
      // Act
      const result = processInput(input)
      
      // Assert
      expect(result).toBe('expected')
    })
  })
})
```

## Resources

- [Vue Testing Guide](https://vuejs.org/guide/scaling-up/testing.html) — Official Vue testing guide

---

> 📘 *This lesson is part of the [Vue Testing Strategies](https://stanza.dev/courses/vue-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*