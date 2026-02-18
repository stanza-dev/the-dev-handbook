---
source_course: "nextjs-testing"
source_lesson: "nextjs-testing-njs-coverage"
---

# Code Coverage

## Introduction

Code coverage tells you what percentage of your code is executed during tests. It's a useful metric, but not the only indicator of test quality.

## Key Concepts

**Coverage types**:

- **Statements**: % of statements executed
- **Branches**: % of if/else branches taken
- **Functions**: % of functions called
- **Lines**: % of lines executed

## Deep Dive

### Enabling Coverage

```json
// package.json
{
  "scripts": {
    "test:coverage": "jest --coverage"
  }
}
```

### Jest Configuration

```javascript
// jest.config.js
module.exports = {
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/*.stories.tsx',
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
};
```

### Interpreting Results

```
----------------------|---------|----------|---------|---------|-------------------
File                  | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s
----------------------|---------|----------|---------|---------|-------------------
All files             |   85.71 |    83.33 |   88.89 |   85.71 |
components/Button.tsx |     100 |      100 |     100 |     100 |
components/Form.tsx   |   71.43 |    66.67 |   77.78 |   71.43 | 23-27,45
----------------------|---------|----------|---------|---------|-------------------
```

## Summary

Coverage helps identify untested code but doesn't guarantee test quality. Aim for meaningful coverage (80%+) and focus on testing critical paths rather than chasing 100%.

## Resources

- [Jest Coverage](https://jestjs.io/docs/configuration#collectcoveragefrom-array) — Jest coverage configuration

---

> 📘 *This lesson is part of the [Next.js Testing Strategies](https://stanza.dev/courses/nextjs-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*