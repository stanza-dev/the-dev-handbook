---
source_course: "react-testing"
source_lesson: "react-testing-testing-setup"
---

# Setting Up Your Testing Environment

Let's configure a modern React testing environment with Jest and React Testing Library.

## Required Packages

```bash
npm install --save-dev \
  @testing-library/react \
  @testing-library/jest-dom \
  @testing-library/user-event \
  jest \
  jest-environment-jsdom
```

## Jest Configuration

```js
// jest.config.js
module.exports = {
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
  moduleNameMapper: {
    // Handle CSS imports
    '^.+\\.css$': 'identity-obj-proxy',
    // Handle module aliases
    '^@/(.*)$': '<rootDir>/src/$1'
  },
  transform: {
    '^.+\\.(js|jsx|ts|tsx)$': ['babel-jest', { presets: ['next/babel'] }]
  }
};
```

## Setup File

```js
// jest.setup.js
import '@testing-library/jest-dom';

// Clean up after each test
afterEach(() => {
  jest.clearAllMocks();
});
```

## The @testing-library/jest-dom Matchers

This package adds helpful matchers:

```jsx
// Element presence
expect(element).toBeInTheDocument();
expect(element).not.toBeInTheDocument();

// Visibility
expect(element).toBeVisible();
expect(element).not.toBeVisible();

// Form state
expect(input).toBeDisabled();
expect(input).toBeEnabled();
expect(input).toBeRequired();
expect(input).toHaveValue('hello');
expect(checkbox).toBeChecked();

// Content
expect(element).toHaveTextContent('Hello');
expect(element).toContainHTML('<span>test</span>');

// Accessibility
expect(input).toHaveAccessibleName('Email');
expect(button).toHaveFocus();

// Classes and styles
expect(element).toHaveClass('active');
expect(element).toHaveStyle({ display: 'flex' });
```

## Project Structure

```
src/
├── components/
│   ├── Button/
│   │   ├── Button.jsx
│   │   ├── Button.test.jsx    # Co-located test
│   │   └── index.js
│   └── Form/
│       ├── Form.jsx
│       └── Form.test.jsx
├── hooks/
│   ├── useAuth.js
│   └── useAuth.test.js
└── __tests__/                  # Integration tests
    └── user-flow.test.jsx
```

## Running Tests

```bash
# Run all tests
npm test

# Run in watch mode
npm test -- --watch

# Run specific file
npm test -- Button.test.jsx

# Run with coverage
npm test -- --coverage
```

---

> 📘 *This lesson is part of the [React Testing Strategies](https://stanza.dev/courses/react-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*