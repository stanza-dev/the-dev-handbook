---
source_course: "nestjs-techniques"
source_lesson: "nestjs-techniques-events"
---

# Event Emitter

## Introduction

Events decouple components. Instead of directly calling methods, emit events that listeners handle. This enables loose coupling, easier testing, and simpler addition of new behaviors without modifying existing code.

## Key Concepts

- **EventEmitter2**: Event library with wildcards and async support
- **@OnEvent()**: Decorator for event listeners
- **Emit**: Publish an event
- **Wildcards**: Match multiple event types with patterns

## Deep Dive

### Setup

```bash
npm install @nestjs/event-emitter
```

```typescript
import { EventEmitterModule } from '@nestjs/event-emitter';

@Module({
  imports: [EventEmitterModule.forRoot()],
})
export class AppModule {}
```

### Emitting Events

```typescript
import { EventEmitter2 } from '@nestjs/event-emitter';

@Injectable()
export class UsersService {
  constructor(private eventEmitter: EventEmitter2) {}

  async createUser(dto: CreateUserDto): Promise<User> {
    const user = await this.userRepository.save(dto);
    
    this.eventEmitter.emit('user.created', {
      userId: user.id,
      email: user.email,
    });
    
    return user;
  }
}
```

### Event Listeners

```typescript
import { OnEvent } from '@nestjs/event-emitter';

@Injectable()
export class EmailListener {
  @OnEvent('user.created')
  handleUserCreated(payload: { userId: string; email: string }) {
    console.log(\`Sending welcome email to \${payload.email}\`);
  }
}

@Injectable()
export class AnalyticsListener {
  @OnEvent('user.created')
  trackUserCreation(payload: { userId: string }) {
    this.analytics.track('user_created', { userId: payload.userId });
  }
}
```

### Wildcard Listeners

```typescript
@OnEvent('user.*')
handleUserEvents(payload: any) {
  console.log('User event:', payload);
}

@OnEvent('**')
handleAllEvents(payload: any) {
  console.log('Any event:', payload);
}
```

### Async Listeners

```typescript
@OnEvent('order.created', { async: true })
async handleOrderCreated(payload: OrderCreatedEvent) {
  await this.emailService.sendOrderConfirmation(payload);
}
```

## Common Pitfalls

1. **Circular dependencies**: Listeners and emitters importing each other. Use forwardRef or events.
2. **Unhandled errors**: Listener errors don't bubble up. Add error handling.
3. **Order dependency**: Don't rely on listener execution order.

## Best Practices

- Define event payload types for type safety
- Use namespaced event names (user.created, order.completed)
- Handle errors in listeners
- Use async: true for slow operations

## Summary

Event emitter enables loose coupling through pub/sub patterns. Emit events with EventEmitter2, listen with @OnEvent(). Use wildcards for broad listeners and async for slow operations.

## Resources

- [Events](https://docs.nestjs.com/techniques/events) — Official events guide

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*