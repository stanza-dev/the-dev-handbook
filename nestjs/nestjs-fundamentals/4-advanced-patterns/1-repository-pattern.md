---
source_course: "nestjs-fundamentals"
source_lesson: "nestjs-fundamentals-repository-pattern"
---

# Repository Pattern

## Introduction

The Repository pattern abstracts data access, separating business logic from database operations. This improves testability, enables switching data sources, and keeps your services focused on business rules rather than SQL queries.

## Key Concepts

- **Repository**: Abstraction layer between domain and data mapping
- **Data Mapper**: Transforms between domain objects and database records
- **Unit of Work**: Groups operations into transactions
- **Specification Pattern**: Encapsulates query criteria

## Real World Context

Repository pattern enables:
- Swapping PostgreSQL for MongoDB without changing services
- Unit testing services with mock repositories
- Centralizing complex queries
- Implementing caching at the data layer

## Deep Dive

### Abstract Repository Interface

```typescript
export interface IRepository<T> {
  findById(id: string): Promise<T | null>;
  findAll(): Promise<T[]>;
  findOne(criteria: Partial<T>): Promise<T | null>;
  create(entity: Partial<T>): Promise<T>;
  update(id: string, entity: Partial<T>): Promise<T>;
  delete(id: string): Promise<void>;
}
```

### TypeORM Repository Implementation

```typescript
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './user.entity';

export abstract class BaseRepository<T> implements IRepository<T> {
  constructor(protected readonly repository: Repository<T>) {}

  async findById(id: string): Promise<T | null> {
    return this.repository.findOne({ where: { id } as any });
  }

  async findAll(): Promise<T[]> {
    return this.repository.find();
  }

  async create(entity: Partial<T>): Promise<T> {
    const created = this.repository.create(entity as any);
    return this.repository.save(created);
  }

  async update(id: string, entity: Partial<T>): Promise<T> {
    await this.repository.update(id, entity as any);
    return this.findById(id);
  }

  async delete(id: string): Promise<void> {
    await this.repository.delete(id);
  }
}

@Injectable()
export class UsersRepository extends BaseRepository<User> {
  constructor(
    @InjectRepository(User)
    repository: Repository<User>,
  ) {
    super(repository);
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.repository.findOne({ where: { email } });
  }

  async findActiveUsers(): Promise<User[]> {
    return this.repository.find({ where: { isActive: true } });
  }
}
```

### Service Using Repository

```typescript
@Injectable()
export class UsersService {
  constructor(private readonly usersRepository: UsersRepository) {}

  async createUser(dto: CreateUserDto): Promise<User> {
    const existing = await this.usersRepository.findByEmail(dto.email);
    if (existing) {
      throw new ConflictException('Email already exists');
    }
    return this.usersRepository.create(dto);
  }

  async getActiveUsers(): Promise<User[]> {
    return this.usersRepository.findActiveUsers();
  }
}
```

### Specification Pattern for Queries

```typescript
export interface Specification<T> {
  isSatisfiedBy(entity: T): boolean;
  toQuery(): FindOptionsWhere<T>;
}

export class ActiveUserSpecification implements Specification<User> {
  isSatisfiedBy(user: User): boolean {
    return user.isActive === true;
  }

  toQuery(): FindOptionsWhere<User> {
    return { isActive: true };
  }
}

export class AdminUserSpecification implements Specification<User> {
  isSatisfiedBy(user: User): boolean {
    return user.role === 'admin';
  }

  toQuery(): FindOptionsWhere<User> {
    return { role: 'admin' };
  }
}

// In repository
async findBySpecification(spec: Specification<User>): Promise<User[]> {
  return this.repository.find({ where: spec.toQuery() });
}
```

## Common Pitfalls

1. **Leaking ORM details**: Repository should return domain objects, not ORM entities.
2. **Too many methods**: Start minimal, add methods as needed.
3. **Business logic in repository**: Keep business rules in services.

## Best Practices

- Define repository interface for testability
- Use base repository for common operations
- Add domain-specific methods as needed
- Return domain objects, not ORM entities
- Use specifications for complex queries

## Summary

The Repository pattern abstracts data access behind an interface. Implement with TypeORM repositories, add domain-specific methods, and use specifications for complex queries. This enables testing, flexibility, and separation of concerns.

## Resources

- [Database](https://docs.nestjs.com/techniques/database) — Database patterns and TypeORM

---

> 📘 *This lesson is part of the [NestJS Fundamentals](https://stanza.dev/courses/nestjs-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*