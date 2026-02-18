---
source_course: "nextjs-testing"
source_lesson: "nextjs-testing-njs-testing-database"
---

# Testing with a Test Database

## Introduction

For integration tests, you sometimes need a real database. This requires setup, teardown, and isolation between tests.

## Key Concepts

**Test database strategies**:

- **In-memory**: SQLite for fast, isolated tests
- **Docker**: Spin up real database for CI
- **Transactions**: Roll back after each test

## Deep Dive

### Using Test Database

```typescript
// jest.setup.ts
import { prisma } from '@/lib/prisma';

beforeAll(async () => {
  // Connect to test database
  await prisma.$connect();
});

beforeEach(async () => {
  // Clean tables before each test
  await prisma.user.deleteMany();
  await prisma.post.deleteMany();
});

afterAll(async () => {
  await prisma.$disconnect();
});
```

### Environment Configuration

```bash
# .env.test
DATABASE_URL="postgresql://test:test@localhost:5433/test_db"
```

```json
// package.json
{
  "scripts": {
    "test:integration": "dotenv -e .env.test -- jest --config jest.integration.config.js"
  }
}
```

### Docker Compose for Tests

```yaml
# docker-compose.test.yml
services:
  test-db:
    image: postgres:15
    environment:
      POSTGRES_USER: test
      POSTGRES_PASSWORD: test
      POSTGRES_DB: test_db
    ports:
      - "5433:5432"
```

## Summary

Use a separate test database for integration tests. Clean data between tests, use Docker for CI, and consider transactions for faster cleanup.

## Resources

- [Prisma Testing](https://www.prisma.io/docs/guides/testing/unit-testing) — Testing with Prisma

---

> 📘 *This lesson is part of the [Next.js Testing Strategies](https://stanza.dev/courses/nextjs-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*