---
source_course: "nextjs-testing"
source_lesson: "nextjs-testing-njs-jest-setup"
---

# Testing Setup with Jest

Jest is the most popular testing framework for JavaScript and integrates well with Next.js.

## Installation

```bash
npm install -D jest jest-environment-jsdom @testing-library/react @testing-library/jest-dom
```

## Configuration

Create `jest.config.js`:

```javascript
const nextJest = require('next/jest');

const createJestConfig = nextJest({
  dir: './', // Path to Next.js app
});

const config = {
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
  testEnvironment: 'jest-environment-jsdom',
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
};

module.exports = createJestConfig(config);
```

Create `jest.setup.js`:

```javascript
import '@testing-library/jest-dom';
```

## Package.json Script

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch"
  }
}
```

The `next/jest` helper automatically handles:
- SWC/Babel transformation
- CSS/image mocking
- Module path aliases

## Resources

- [Testing Documentation](https://nextjs.org/docs/app/building-your-application/testing/jest) — Official Jest setup guide

---

> 📘 *This lesson is part of the [Next.js Testing Strategies](https://stanza.dev/courses/nextjs-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*