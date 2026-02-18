---
source_course: "nextjs-testing"
source_lesson: "nextjs-testing-njs-test-structure"
---

# Test File Organization

## Introduction

Well-organized tests are easier to find, maintain, and run. Establishing conventions early saves headaches later.

## Key Concepts

**Common patterns**:

- **Colocated**: Test files next to source files
- **Separate directory**: All tests in `__tests__/` or `tests/`
- **Hybrid**: Unit tests colocated, integration tests separate

## Deep Dive

### Colocation Pattern

```
src/
  components/
    Button/
      Button.tsx
      Button.test.tsx
      Button.stories.tsx
    Form/
      Form.tsx
      Form.test.tsx
```

### Test Naming

```typescript
// Button.test.tsx
describe('Button', () => {
  describe('when clicked', () => {
    it('calls onClick handler', () => { /* ... */ });
    it('shows loading state when pending', () => { /* ... */ });
  });

  describe('when disabled', () => {
    it('does not call onClick handler', () => { /* ... */ });
    it('has disabled attribute', () => { /* ... */ });
  });
});
```

### Excluding from Build

```typescript
// next.config.js
module.exports = {
  // Test files won't be included in the build
  pageExtensions: ['page.tsx', 'page.ts'],
};
```

## Summary

Organize tests in a way that's easy to navigate—colocation keeps tests close to source code. Use descriptive test names and exclude test files from production builds.

## Resources

- [Jest Configuration](https://jestjs.io/docs/configuration) — Jest configuration options

---

> 📘 *This lesson is part of the [Next.js Testing Strategies](https://stanza.dev/courses/nextjs-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*