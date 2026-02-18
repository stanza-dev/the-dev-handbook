---
source_course: "nextjs-testing"
source_lesson: "nextjs-testing-njs-testing-hooks"
---

# Testing Custom Hooks

## Introduction

Custom hooks contain reusable logic. Testing them directly—rather than through components—makes tests faster and more focused.

## Key Concepts

**Hook testing rules**:

- Hooks must be called inside a React component
- Use renderHook from @testing-library/react
- Test state changes and return values

## Deep Dive

### Basic Hook Test

```typescript
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

describe('useCounter', () => {
  it('starts at initial value', () => {
    const { result } = renderHook(() => useCounter(5));
    expect(result.current.count).toBe(5);
  });

  it('increments count', () => {
    const { result } = renderHook(() => useCounter(0));
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });
});
```

### Testing with Dependencies

```typescript
import { renderHook } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { useUser } from './useUser';

const wrapper = ({ children }) => {
  const queryClient = new QueryClient();
  return (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
};

describe('useUser', () => {
  it('fetches user data', async () => {
    const { result } = renderHook(() => useUser('123'), { wrapper });
    
    await waitFor(() => {
      expect(result.current.data).toBeDefined();
    });
  });
});
```

## Summary

Test hooks with renderHook(), wrap state changes in act(), and provide necessary context via the wrapper option. This isolates hook logic from component rendering.

## Resources

- [Testing Hooks](https://testing-library.com/docs/react-testing-library/api#renderhook) — renderHook API documentation

---

> 📘 *This lesson is part of the [Next.js Testing Strategies](https://stanza.dev/courses/nextjs-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*