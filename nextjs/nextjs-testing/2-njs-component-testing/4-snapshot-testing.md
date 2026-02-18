---
source_course: "nextjs-testing"
source_lesson: "nextjs-testing-njs-snapshot-testing"
---

# Snapshot Testing

## Introduction

Snapshot tests capture a component's output and compare it against future renders. They're great for catching unintended UI changes but can become maintenance burdens.

## Key Concepts

**Snapshot testing**:

- Captures rendered output as a file
- Fails when output changes
- Update with `jest -u` when changes are intentional

## Deep Dive

### Basic Snapshot

```typescript
import { render } from '@testing-library/react';
import { Button } from './Button';

it('matches snapshot', () => {
  const { container } = render(<Button>Click me</Button>);
  expect(container).toMatchSnapshot();
});
```

### Inline Snapshots

```typescript
it('renders correctly', () => {
  const { container } = render(<Button>Click</Button>);
  expect(container).toMatchInlineSnapshot(`
    <div>
      <button class="btn btn-primary">
        Click
      </button>
    </div>
  `);
});
```

### When to Use Snapshots

**Good for**:
- Small, stable components
- Catching accidental changes
- Quick initial test coverage

**Bad for**:
- Large, complex components
- Components with dynamic content
- Testing behavior (use assertions instead)

## Summary

Snapshots catch unintended changes but require maintenance. Use them sparingly for small, stable components. Prefer behavior-based assertions for testing functionality.

## Resources

- [Snapshot Testing](https://jestjs.io/docs/snapshot-testing) — Jest snapshot testing guide

---

> 📘 *This lesson is part of the [Next.js Testing Strategies](https://stanza.dev/courses/nextjs-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*