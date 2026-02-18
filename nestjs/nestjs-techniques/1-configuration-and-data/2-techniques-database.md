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

## Deep Dive

### TypeORM Setup

```bash
npm install @nestjs/typeorm typeorm pg
```

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

### Entity Definition

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

### Repository Pattern

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

### Async Configuration

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

NestJS integrates with TypeORM, Prisma, and other ORMs through dedicated modules. Use forRoot/forRootAsync for configuration, forFeature for entity registration, and repository pattern for data access.

## Resources

- [Database](https://docs.nestjs.com/techniques/database) — Official database guide

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*