---
source_course: "nestjs-techniques"
source_lesson: "nestjs-techniques-logger"
---

# Logging

## Introduction

Logging is essential for debugging, monitoring, and auditing. NestJS provides a built-in Logger that can be replaced with popular libraries like Winston or Pino for production needs.

## Key Concepts

- **Logger**: Built-in NestJS logging class
- **Log Levels**: error, warn, log, debug, verbose
- **Context**: Identifies which component logged the message
- **Custom Logger**: Replace built-in with external libraries

## Deep Dive

### Built-in Logger

```typescript
import { Logger, Injectable } from '@nestjs/common';

@Injectable()
export class UsersService {
  private readonly logger = new Logger(UsersService.name);

  create(dto: CreateUserDto) {
    this.logger.log(\`Creating user: \${dto.email}\`);
    this.logger.debug('Debug details here');
    this.logger.warn('This might be a problem');
    this.logger.error('Something went wrong', error.stack);
  }
}
```

### Global Logger Configuration

```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule, {
    logger: ['error', 'warn', 'log'], // Only these levels
  });
}

// Or disable entirely
const app = await NestFactory.create(AppModule, {
  logger: false,
});
```

### Custom Logger (Winston)

```typescript
import { LoggerService } from '@nestjs/common';
import * as winston from 'winston';

export class WinstonLogger implements LoggerService {
  private logger: winston.Logger;

  constructor() {
    this.logger = winston.createLogger({
      level: 'info',
      format: winston.format.json(),
      transports: [
        new winston.transports.File({ filename: 'error.log', level: 'error' }),
        new winston.transports.File({ filename: 'combined.log' }),
      ],
    });
  }

  log(message: string, context?: string) {
    this.logger.info(message, { context });
  }

  error(message: string, trace?: string, context?: string) {
    this.logger.error(message, { trace, context });
  }

  warn(message: string, context?: string) {
    this.logger.warn(message, { context });
  }

  debug(message: string, context?: string) {
    this.logger.debug(message, { context });
  }
}

// Use in bootstrap
app.useLogger(new WinstonLogger());
```

### Request Logging

```typescript
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  private readonly logger = new Logger('HTTP');

  intercept(context: ExecutionContext, next: CallHandler) {
    const request = context.switchToHttp().getRequest();
    const { method, url } = request;
    const start = Date.now();

    return next.handle().pipe(
      tap(() => {
        const duration = Date.now() - start;
        this.logger.log(\`\${method} \${url} - \${duration}ms\`);
      }),
    );
  }
}
```

## Common Pitfalls

1. **Logging sensitive data**: Never log passwords, tokens, or PII.
2. **Performance**: Excessive logging impacts performance. Use appropriate levels.
3. **Lost context**: Include request IDs for tracing in distributed systems.

## Best Practices

- Use structured logging (JSON) for production
- Include correlation IDs for request tracing
- Use appropriate log levels
- Never log sensitive information

## Summary

NestJS Logger provides contextual logging with multiple levels. Replace with Winston or Pino for production features like file output and structured logging. Include correlation IDs for distributed tracing.

## Resources

- [Logger](https://docs.nestjs.com/techniques/logger) — Official logger guide

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*