---
source_course: "nestjs-performance"
source_lesson: "nestjs-performance-microservices"
---

# Microservices Introduction

## Introduction

As applications grow, monoliths become hard to scale and maintain. Microservices split functionality into independently deployable services that communicate over networks. NestJS provides first-class support for various transport layers.

## Key Concepts

- **Transport Layer**: Communication protocol (TCP, Redis, NATS, etc.)
- **Client**: Service that sends messages
- **Server/Microservice**: Service that receives and handles messages
- **Message Pattern**: Request/response or event-based communication

## Deep Dive

### Creating a Microservice

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { Transport, MicroserviceOptions } from '@nestjs/microservices';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.TCP,
      options: { host: '0.0.0.0', port: 3001 },
    },
  );
  await app.listen();
}
bootstrap();
```

### Message Handler

```typescript
import { Controller } from '@nestjs/common';
import { MessagePattern, EventPattern, Payload } from '@nestjs/microservices';

@Controller()
export class AppController {
  // Request/response pattern
  @MessagePattern({ cmd: 'sum' })
  sum(@Payload() data: number[]): number {
    return data.reduce((a, b) => a + b, 0);
  }

  // Event pattern (fire and forget)
  @EventPattern('user_created')
  handleUserCreated(@Payload() data: { userId: string }) {
    console.log('User created:', data.userId);
  }
}
```

### Client Setup

```typescript
import { ClientsModule, Transport } from '@nestjs/microservices';

@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'MATH_SERVICE',
        transport: Transport.TCP,
        options: { host: 'localhost', port: 3001 },
      },
    ]),
  ],
})
export class AppModule {}
```

### Calling a Microservice

```typescript
import { Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';
import { firstValueFrom } from 'rxjs';

@Injectable()
export class CalcService {
  constructor(@Inject('MATH_SERVICE') private client: ClientProxy) {}

  // Request/response
  async sum(numbers: number[]): Promise<number> {
    return firstValueFrom(
      this.client.send<number>({ cmd: 'sum' }, numbers),
    );
  }

  // Event (fire and forget)
  emitUserCreated(userId: string) {
    this.client.emit('user_created', { userId });
  }
}
```

### Transport Options

| Transport | Use Case |
|-----------|----------|
| TCP | Simple, direct communication |
| Redis | Pub/sub with persistence |
| NATS | High-performance messaging |
| RabbitMQ | Enterprise messaging with queues |
| Kafka | High-throughput event streaming |
| gRPC | Strongly-typed, efficient protocol |

## Common Pitfalls

1. **Network failures**: Implement retry logic and circuit breakers.
2. **Serialization**: Ensure data can be serialized across the network.
3. **Distributed debugging**: Add correlation IDs for tracing.

## Best Practices

- Start with TCP, move to Redis/NATS when needed
- Use events for fire-and-forget, messages for request/response
- Implement health checks between services
- Add circuit breakers for resilience

## Summary

NestJS microservices communicate via various transports. Use @MessagePattern for request/response, @EventPattern for events. Register clients with ClientsModule and call services with ClientProxy. Choose transport based on your scaling needs.

## Resources

- [Microservices](https://docs.nestjs.com/microservices/basics) — Official microservices guide

---

> 📘 *This lesson is part of the [NestJS High Performance](https://stanza.dev/courses/nestjs-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*