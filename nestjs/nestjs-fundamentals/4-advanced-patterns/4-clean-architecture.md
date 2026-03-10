---
source_course: "nestjs-fundamentals"
source_lesson: "nestjs-fundamentals-clean-architecture"
---

# Clean Architecture

## Introduction

Clean Architecture organizes code in layers with dependencies pointing inward. The domain layer is independent of frameworks, databases, and delivery mechanisms. This makes the core business logic testable and maintainable.

## Key Concepts

- **Dependency Rule**: Inner layers don't know about outer layers
- **Use Cases**: Application-specific business rules
- **Interface Adapters**: Convert data between use cases and external systems
- **Frameworks & Drivers**: External tools and frameworks (outermost layer)

## Real World Context

Clean Architecture enables:
- Swapping frameworks without touching business logic
- Testing business rules in isolation
- Independent deployability of components
- Clear boundaries between concerns

## Deep Dive

### Layer Structure

```
src/
├── domain/              # Enterprise Business Rules
│   ├── entities/
│   └── value-objects/
├── application/         # Application Business Rules
│   ├── use-cases/
│   ├── ports/           # Interfaces
│   └── dtos/
├── infrastructure/      # Interface Adapters & Frameworks
│   ├── persistence/
│   ├── http/
│   └── external-services/
└── main.ts              # Composition root
```

### Domain Layer (Innermost)

```typescript
// domain/entities/user.entity.ts
export class User {
  constructor(
    public readonly id: string,
    private _email: string,
    private _name: string,
  ) {}

  changeName(newName: string): void {
    if (!newName || newName.length < 2) {
      throw new Error('Invalid name');
    }
    this._name = newName;
  }

  get email(): string { return this._email; }
  get name(): string { return this._name; }
}
```

### Application Layer (Use Cases)

```typescript
// application/ports/user.repository.port.ts
export interface UserRepositoryPort {
  findById(id: string): Promise<User | null>;
  save(user: User): Promise<void>;
}

// application/use-cases/create-user.use-case.ts
export class CreateUserUseCase {
  constructor(
    private userRepository: UserRepositoryPort,
    private passwordHasher: PasswordHasherPort,
  ) {}

  async execute(input: CreateUserInput): Promise<CreateUserOutput> {
    const hashedPassword = await this.passwordHasher.hash(input.password);
    
    const user = new User(
      uuid(),
      input.email,
      input.name,
    );
    
    await this.userRepository.save(user);
    
    return { userId: user.id };
  }
}
```

### Infrastructure Layer (Adapters)

```typescript
// infrastructure/persistence/typeorm-user.repository.ts
@Injectable()
export class TypeOrmUserRepository implements UserRepositoryPort {
  constructor(
    @InjectRepository(UserOrmEntity)
    private ormRepository: Repository<UserOrmEntity>,
  ) {}

  async findById(id: string): Promise<User | null> {
    const entity = await this.ormRepository.findOne({ where: { id } });
    return entity ? this.toDomain(entity) : null;
  }

  async save(user: User): Promise<void> {
    const entity = this.toOrmEntity(user);
    await this.ormRepository.save(entity);
  }

  private toDomain(entity: UserOrmEntity): User {
    return new User(entity.id, entity.email, entity.name);
  }

  private toOrmEntity(user: User): UserOrmEntity {
    return { id: user.id, email: user.email, name: user.name };
  }
}
```

### HTTP Layer (Controllers)

```typescript
// infrastructure/http/users.controller.ts
@Controller('users')
export class UsersController {
  constructor(
    @Inject('CreateUserUseCase')
    private createUserUseCase: CreateUserUseCase,
  ) {}

  @Post()
  async create(@Body() dto: CreateUserDto) {
    return this.createUserUseCase.execute({
      email: dto.email,
      name: dto.name,
      password: dto.password,
    });
  }
}
```

### Dependency Injection Configuration

```typescript
@Module({
  providers: [
    {
      provide: 'UserRepositoryPort',
      useClass: TypeOrmUserRepository,
    },
    {
      provide: 'CreateUserUseCase',
      useFactory: (repo: UserRepositoryPort, hasher: PasswordHasherPort) =>
        new CreateUserUseCase(repo, hasher),
      inject: ['UserRepositoryPort', 'PasswordHasherPort'],
    },
  ],
})
export class UsersModule {}
```

## Common Pitfalls

1. **Skipping layers**: Controller directly calling repository bypasses use cases.
2. **Domain depending on ORM**: Domain entities should be plain TypeScript.
3. **Overengineering**: Simple CRUD doesn't need Clean Architecture.

## Best Practices

- Domain layer has no external dependencies
- Use cases orchestrate domain logic
- Ports (interfaces) define boundaries
- Adapters implement ports for specific technologies
- Test use cases with mock adapters

## Summary

Clean Architecture separates concerns into layers. Domain contains business rules, application contains use cases, infrastructure contains adapters. Dependencies point inward. Use for complex systems requiring flexibility and testability.

## Code Examples

**Clean Architecture ports and adapters — the domain defines interfaces, infrastructure implements them, dependencies point inward**

```typescript
// Port (interface) — domain layer
export interface UserRepositoryPort {
  findById(id: string): Promise<User | null>;
  save(user: User): Promise<void>;
}

// Adapter (implementation) — infrastructure layer
@Injectable()
export class PrismaUserRepository implements UserRepositoryPort {
  constructor(private prisma: PrismaService) {}
  async findById(id: string) { return this.prisma.user.findUnique({ where: { id } }); }
  async save(user: User) { await this.prisma.user.upsert({ ... }); }
}
```


## Resources

- [Dynamic Modules](https://docs.nestjs.com/fundamentals/dynamic-modules) — Building modular, composable architectures with dynamic modules

---

> 📘 *This lesson is part of the [NestJS Fundamentals](https://stanza.dev/courses/nestjs-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*