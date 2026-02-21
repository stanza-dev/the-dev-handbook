---
source_course: "nextjs-testing"
source_lesson: "nextjs-testing-njs-testing-middleware"
---

# Testing Proxy

## Introduction

The Proxy handles authentication, redirects, and request modification. Testing it requires creating mock requests and verifying responses.

## Key Concepts

**Proxy testing**:

- Create mock NextRequest objects
- Call proxy function directly
- Assert on NextResponse properties

## Deep Dive

### Testing Redirects

```typescript
import { NextRequest } from 'next/server';
import { proxy } from '../proxy';

describe('proxy', () => {
  it('redirects unauthenticated users to login', async () => {
    const request = new NextRequest(
      new URL('http://localhost:3000/dashboard')
    );

    const response = await proxy(request);

    expect(response.status).toBe(307);
    expect(response.headers.get('location')).toContain('/login');
  });

  it('allows authenticated users', async () => {
    const request = new NextRequest(
      new URL('http://localhost:3000/dashboard'),
      {
        headers: {
          cookie: 'session=valid-token',
        },
      }
    );

    const response = await proxy(request);

    expect(response.status).not.toBe(307);
  });
});
```

### Mocking JWT Verification

```typescript
jest.mock('jose', () => ({
  jwtVerify: jest.fn(),
}));

import { jwtVerify } from 'jose';

beforeEach(() => {
  (jwtVerify as jest.Mock).mockResolvedValue({
    payload: { userId: '123', role: 'user' },
  });
});
```

## Summary

Test the proxy by creating mock requests with NextRequest, calling your proxy function, and asserting on the response. Mock external dependencies like JWT verification.

## Resources

- [NextRequest](https://nextjs.org/docs/app/api-reference/functions/next-request) — NextRequest API reference

---

> 📘 *This lesson is part of the [Next.js Testing Strategies](https://stanza.dev/courses/nextjs-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*