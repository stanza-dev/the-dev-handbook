---
source_course: "nestjs-techniques"
source_lesson: "nestjs-techniques-database"
---

# Database Integration

## Introduction

Most applications need a database. NestJS integrates seamlessly with SQL databases (via TypeORM, Prisma, Sequelize) and NoSQL databases (via Mongoose, Mikro-ORM). Understanding these integrations is essential for building data-driven applications.

## Key Concepts

- **TypeORM**: Popular ORM for SQL databases
- **Prisma**: Modern ORM with type-safe queries
- **Repository Pattern**: Abstraction layer for data access
- **forRootAsync**: Async configuration for database modules

## Real World Context

Most production applications are data-driven. Whether you're building an e-commerce platform storing product catalogs, a SaaS app managing user accounts, or an analytics service processing event streams, you need reliable database access. Choosing the right ORM and configuring it properly can save weeks of debugging connection issues in production.

## Deep Dive

### TypeORM Setup

Install the required packages and configure `TypeOrmModule` in your root module with connection details.

```bash
npm install @nestjs/typeorm typeorm pg
```

Register the module with `forRoot()` and pass your connection options. Note that `synchronize: true` should only be used in development.

```typescript
@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      username: 'user',
      password: 'password',
      database: 'mydb',
      entities: [User],
      synchronize: true, // Only for development!
    }),
  ],
})
export class AppModule {}
```

This establishes a connection pool that persists for the application's lifetime.

### Entity Definition

Entities map TypeScript classes to database tables using TypeORM decorators.

```typescript
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  email: string;

  @Column()
  name: string;

  @Column({ default: true })
  isActive: boolean;
}
```

Each `@Column()` maps to a database column, and options like `unique` and `default` translate directly to SQL constraints.

### Repository Pattern

Register entities per feature module with `forFeature()`, then inject the auto-generated repository.

```typescript
// users.module.ts
@Module({
  imports: [TypeOrmModule.forFeature([User])],
  providers: [UsersService],
})
export class UsersModule {}

// users.service.ts
@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private usersRepository: Repository<User>,
  ) {}

  findAll(): Promise<User[]> {
    return this.usersRepository.find();
  }

  findOne(id: string): Promise<User> {
    return this.usersRepository.findOneBy({ id });
  }

  async create(dto: CreateUserDto): Promise<User> {
    const user = this.usersRepository.create(dto);
    return this.usersRepository.save(user);
  }
}
```

The `create()` method builds an entity instance without persisting, while `save()` writes it to the database.

### Async Configuration

Use `forRootAsync()` when database settings come from `ConfigService` or other injected providers.

```typescript
TypeOrmModule.forRootAsync({
  imports: [ConfigModule],
  useFactory: (configService: ConfigService) => ({
    type: 'postgres',
    url: configService.get('DATABASE_URL'),
    autoLoadEntities: true,
    synchronize: configService.get('NODE_ENV') !== 'production',
  }),
  inject: [ConfigService],
})
```

The `autoLoadEntities` option automatically registers every entity imported via `forFeature()`, eliminating the need to list them manually.

## Common Pitfalls

1. **synchronize: true in production**: This can cause data loss. Use migrations instead.
2. **Not using transactions**: Multiple related operations should be wrapped in transactions.
3. **N+1 queries**: Eager load relations or use query builder for complex queries.

## Best Practices

- Use migrations for schema changes in production
- Implement repository abstraction for testability
- Use forRootAsync for environment-based configuration
- Enable query logging in development

## Summary

- NestJS integrates with TypeORM, Prisma, and other ORMs through dedicated modules
- Use forRoot() for static config and forRootAsync() for environment-based configuration
- Register entities per module with TypeOrmModule.forFeature()
- Use the repository pattern for testable data access
- Always use migrations instead of synchronize in production

## Code Examples

**Setting up TypeORM with PostgreSQL and configuring entity auto-loading**

```typescript
@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      username: 'user',
      password: 'password',
      database: 'mydb',
      entities: [User],
      synchronize: true, // Only for development!
    }),
  ],
})
export class AppModule {}
```


## Resources

- [Database](https://docs.nestjs.com/techniques/database) — Official database guide

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*