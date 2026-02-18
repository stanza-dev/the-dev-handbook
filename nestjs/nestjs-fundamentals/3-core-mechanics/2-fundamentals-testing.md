---
source_course: "nestjs-fundamentals"
source_lesson: "nestjs-fundamentals-testing"
---

# Testing

## Introduction

NestJS is built with testing in mind. Its dependency injection system makes it trivial to mock services and test components in isolation. The framework provides testing utilities that mirror the production DI container.

## Key Concepts

- **Test.createTestingModule()**: Creates a test module with mock providers
- **overrideProvider()**: Replace real implementations with mocks
- **compile()**: Build the test module
- **Unit vs E2E**: Test isolation levels

## Real World Context

Testing is non-negotiable in production codebases. NestJS makes it easy to:
- Unit test services with mocked dependencies
- Integration test modules together
- E2E test entire request flows

## Deep Dive

### Unit Testing a Service

```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { UsersService } from './users.service';
import { UsersRepository } from './users.repository';

describe('UsersService', () => {
  let service: UsersService;
  let mockRepository: Partial<UsersRepository>;

  beforeEach(async () => {
    mockRepository = {
      findOne: jest.fn().mockResolvedValue({ id: '1', name: 'Test' }),
      create: jest.fn().mockImplementation(dto => ({ id: '2', ...dto })),
    };

    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UsersService,
        { provide: UsersRepository, useValue: mockRepository },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
  });

  it('should find a user', async () => {
    const user = await service.findOne('1');
    expect(user).toEqual({ id: '1', name: 'Test' });
    expect(mockRepository.findOne).toHaveBeenCalledWith('1');
  });
});
```

### Using overrideProvider

```typescript
const module = await Test.createTestingModule({
  imports: [UsersModule],
})
  .overrideProvider(UsersRepository)
  .useValue(mockRepository)
  .compile();
```

### E2E Testing

```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { INestApplication } from '@nestjs/common';
import * as request from 'supertest';
import { AppModule } from '../src/app.module';

describe('UsersController (e2e)', () => {
  let app: INestApplication;

  beforeEach(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.init();
  });

  afterEach(async () => {
    await app.close();
  });

  it('/users (GET)', () => {
    return request(app.getHttpServer())
      .get('/users')
      .expect(200)
      .expect(res => {
        expect(Array.isArray(res.body)).toBe(true);
      });
  });
});
```

### Testing with Databases

```typescript
// Use in-memory database for tests
const module = await Test.createTestingModule({
  imports: [
    TypeOrmModule.forRoot({
      type: 'sqlite',
      database: ':memory:',
      entities: [User],
      synchronize: true,
    }),
  ],
}).compile();
```

## Common Pitfalls

1. **Testing implementation, not behavior**: Test what methods do, not how they do it.
2. **Incomplete mocking**: Mock all dependencies or tests become integration tests.
3. **Shared state between tests**: Always reset state in beforeEach.

## Best Practices

- Mock external dependencies (databases, APIs)
- Use factories for test data
- Test edge cases and error scenarios
- Keep unit tests fast (< 100ms each)
- Use E2E tests sparingly for critical flows

## Summary

NestJS testing utilities (Test.createTestingModule, overrideProvider) make unit testing straightforward. Mock dependencies for isolation, use actual modules for integration tests, and Supertest for E2E testing.

## Resources

- [Testing](https://docs.nestjs.com/fundamentals/testing) — Official testing guide

---

> 📘 *This lesson is part of the [NestJS Fundamentals](https://stanza.dev/courses/nestjs-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*