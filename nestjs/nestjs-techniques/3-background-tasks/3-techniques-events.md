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

## Real World Context

Tightly coupled code is hard to maintain. When a user signs up, you need to send a welcome email, create default settings, and log the event. Instead of calling three services from the registration handler, emit a 'user.created' event and let each service react independently.

## Deep Dive

### Setup

Install the event emitter package and register it in your root module.

```bash
npm install @nestjs/event-emitter
```

Calling `forRoot()` initializes the underlying `EventEmitter2` instance with wildcard and async support.

```typescript
import { EventEmitterModule } from '@nestjs/event-emitter';

@Module({
  imports: [EventEmitterModule.forRoot()],
})
export class AppModule {}
```

Once registered, you can inject `EventEmitter2` in any service to emit events.

### Emitting Events

Inject `EventEmitter2` and call `emit()` with a namespaced event name and a payload object.

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

The event fires synchronously by default, so all listeners run before `createUser` returns.

### Event Listeners

Decorate methods with `@OnEvent()` to subscribe to specific events. Multiple listeners can handle the same event.

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

Each listener is independent, so adding new reactions to an event requires no changes to the emitting service.

### Wildcard Listeners

Use `*` to match a single segment or `**` to match any number of segments in event names.

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

Wildcard listeners are useful for cross-cutting concerns like logging or auditing all events.

### Async Listeners

Pass `{ async: true }` to `@OnEvent()` so the listener runs asynchronously and does not block the emitter.

```typescript
@OnEvent('order.created', { async: true })
async handleOrderCreated(payload: OrderCreatedEvent) {
  await this.emailService.sendOrderConfirmation(payload);
}
```

Without `async: true`, an `await` inside the listener would block the emitting method until the email is sent.

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

- EventEmitter2 enables loose coupling through publish/subscribe patterns
- Emit events from services and listen with @OnEvent() in separate listeners
- Use wildcards (e.g., 'user.*') to match multiple event types
- Enable async: true for listeners that perform slow operations
- Define typed event payloads for type safety across emitters and listeners

## Code Examples

**Emitting and listening to application events with EventEmitter2**

```typescript
import { EventEmitterModule } from '@nestjs/event-emitter';

@Module({
  imports: [EventEmitterModule.forRoot()],
})
export class AppModule {}
```


## Resources

- [Events](https://docs.nestjs.com/techniques/events) — Official events guide

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*