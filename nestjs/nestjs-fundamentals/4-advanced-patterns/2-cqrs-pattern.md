---
source_course: "nestjs-fundamentals"
source_lesson: "nestjs-fundamentals-cqrs-pattern"
---

# CQRS Pattern

## Introduction

Command Query Responsibility Segregation (CQRS) separates read and write operations into different models. This enables independent scaling, optimized data stores for each operation type, and cleaner code for complex domains.

## Key Concepts

- **Command**: Writes/mutations that change state
- **Query**: Reads that return data without side effects
- **Command Handler**: Executes a specific command
- **Query Handler**: Executes a specific query

## Real World Context

CQRS benefits:
- Scale reads and writes independently
- Optimize read models for specific views
- Audit trail via command history
- Event sourcing compatibility

## Deep Dive

### Setup

```bash
npm install @nestjs/cqrs
```

```typescript
import { CqrsModule } from '@nestjs/cqrs';

@Module({
  imports: [CqrsModule],
  providers: [...CommandHandlers, ...QueryHandlers],
})
export class UsersModule {}
```

### Command Definition

```typescript
export class CreateUserCommand {
  constructor(
    public readonly email: string,
    public readonly name: string,
    public readonly password: string,
  ) {}
}

export class UpdateUserCommand {
  constructor(
    public readonly userId: string,
    public readonly name: string,
  ) {}
}
```

### Command Handler

```typescript
import { CommandHandler, ICommandHandler } from '@nestjs/cqrs';

@CommandHandler(CreateUserCommand)
export class CreateUserHandler implements ICommandHandler<CreateUserCommand> {
  constructor(
    private usersRepository: UsersRepository,
    private eventBus: EventBus,
  ) {}

  async execute(command: CreateUserCommand): Promise<string> {
    const { email, name, password } = command;
    
    const hashedPassword = await bcrypt.hash(password, 10);
    const user = await this.usersRepository.create({
      email,
      name,
      password: hashedPassword,
    });
    
    // Publish domain event
    this.eventBus.publish(new UserCreatedEvent(user.id, user.email));
    
    return user.id;
  }
}
```

### Query Definition and Handler

```typescript
export class GetUserByIdQuery {
  constructor(public readonly userId: string) {}
}

@QueryHandler(GetUserByIdQuery)
export class GetUserByIdHandler implements IQueryHandler<GetUserByIdQuery> {
  constructor(private usersRepository: UsersRepository) {}

  async execute(query: GetUserByIdQuery): Promise<UserDto> {
    const user = await this.usersRepository.findById(query.userId);
    if (!user) throw new NotFoundException();
    return new UserDto(user);
  }
}
```

### Controller Using CQRS

```typescript
import { CommandBus, QueryBus } from '@nestjs/cqrs';

@Controller('users')
export class UsersController {
  constructor(
    private commandBus: CommandBus,
    private queryBus: QueryBus,
  ) {}

  @Post()
  async createUser(@Body() dto: CreateUserDto) {
    const userId = await this.commandBus.execute(
      new CreateUserCommand(dto.email, dto.name, dto.password),
    );
    return { id: userId };
  }

  @Get(':id')
  async getUser(@Param('id') id: string) {
    return this.queryBus.execute(new GetUserByIdQuery(id));
  }
}
```

### Domain Events

```typescript
export class UserCreatedEvent {
  constructor(
    public readonly userId: string,
    public readonly email: string,
  ) {}
}

@EventsHandler(UserCreatedEvent)
export class UserCreatedHandler implements IEventHandler<UserCreatedEvent> {
  constructor(private emailService: EmailService) {}

  handle(event: UserCreatedEvent) {
    this.emailService.sendWelcomeEmail(event.email);
  }
}
```

## Common Pitfalls

1. **Overcomplicating simple CRUD**: CQRS adds complexity. Use for complex domains.
2. **Commands returning data**: Commands should return IDs at most, not full objects.
3. **Ignoring eventual consistency**: Read models may be stale.

## Best Practices

- Use CQRS for complex domains with different read/write patterns
- Commands should be imperative (CreateUser, UpdateOrder)
- Queries should be declarative (GetUserById, FindActiveOrders)
- Combine with event sourcing for full audit trail
- Start simple, adopt CQRS when complexity warrants it

## Summary

CQRS separates reads and writes into distinct models. Use @nestjs/cqrs for commands and queries. Commands mutate state, queries return data. Combine with domain events for reactive side effects.

## Resources

- [CQRS](https://docs.nestjs.com/recipes/cqrs) — Official CQRS guide

---

> 📘 *This lesson is part of the [NestJS Fundamentals](https://stanza.dev/courses/nestjs-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*