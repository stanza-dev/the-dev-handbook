---
source_course: "nextjs-testing"
source_lesson: "nextjs-testing-njs-test-doubles"
---

# Test Doubles: Mocks, Stubs, Spies

## Introduction

Test doubles replace real dependencies during testing. Understanding the difference between mocks, stubs, and spies helps you choose the right tool.

## Key Concepts

**Types of test doubles**:

- **Stub**: Returns pre-programmed responses
- **Spy**: Records calls while using real implementation
- **Mock**: Stubs with verification of how it was called

## Deep Dive

### Creating Stubs

```typescript
// Stub - just returns a value
const getUserStub = jest.fn().mockResolvedValue({
  id: '1',
  name: 'John',
});
```

### Creating Spies

```typescript
// Spy - calls real implementation and records
const consoleSpy = jest.spyOn(console, 'log');

await doSomething();

expect(consoleSpy).toHaveBeenCalledWith('Expected message');
consoleSpy.mockRestore();
```

### Full Mock with Verification

```typescript
const mockPrisma = {
  user: {
    findUnique: jest.fn(),
    create: jest.fn(),
  },
};

jest.mock('@/lib/prisma', () => ({ prisma: mockPrisma }));

it('creates user with correct data', async () => {
  await createUser({ name: 'John' });
  
  expect(mockPrisma.user.create).toHaveBeenCalledWith({
    data: { name: 'John' },
  });
});
```

### Mock Return Values

```typescript
mock.mockReturnValue(42);           // Sync
mock.mockResolvedValue({ data });   // Async success
mock.mockRejectedValue(new Error());
 mock.mockImplementation((x) => x * 2);
```

## Summary

Use stubs when you only need controlled return values, spies when you want to observe real behavior, and full mocks when you need both controlled behavior and call verification.

## Resources

- [Jest Mock Functions](https://jestjs.io/docs/mock-functions) — Jest mock function API

---

> 📘 *This lesson is part of the [Next.js Testing Strategies](https://stanza.dev/courses/nextjs-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*