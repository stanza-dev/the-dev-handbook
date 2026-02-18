---
source_course: "nextjs-testing"
source_lesson: "nextjs-testing-njs-cypress"
---

# Cypress for E2E Testing

## Introduction

Cypress is another popular E2E testing framework. It's known for its debugging experience and real-time reloading during test development.

## Key Concepts

**Cypress vs Playwright**:

- Cypress: Better debugging, slower, Chromium-focused
- Playwright: Faster, multi-browser, better parallelization

## Deep Dive

### Setup

```bash
npm install -D cypress
npx cypress open
```

### Basic Test

```typescript
// cypress/e2e/home.cy.ts
describe('Home Page', () => {
  it('displays welcome message', () => {
    cy.visit('/');
    cy.contains('Welcome').should('be.visible');
  });

  it('navigates to about page', () => {
    cy.visit('/');
    cy.get('a[href="/about"]').click();
    cy.url().should('include', '/about');
  });
});
```

### Cypress Configuration for Next.js

```javascript
// cypress.config.ts
import { defineConfig } from 'cypress';

export default defineConfig({
  e2e: {
    baseUrl: 'http://localhost:3000',
    supportFile: false,
  },
});
```

## Summary

Cypress offers excellent debugging with its time-travel feature. Choose Cypress for its developer experience during test writing, or Playwright for speed and cross-browser testing.

## Resources

- [Cypress with Next.js](https://nextjs.org/docs/app/building-your-application/testing/cypress) — Setting up Cypress with Next.js

---

> 📘 *This lesson is part of the [Next.js Testing Strategies](https://stanza.dev/courses/nextjs-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*