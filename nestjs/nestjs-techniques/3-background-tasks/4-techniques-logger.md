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

## Real World Context

Console.log in production is a debugging nightmare—no timestamps, no log levels, no structured output. A proper logger enables filtering by severity, adds context (which module, which request), and integrates with log aggregation services like Datadog or CloudWatch.

## Deep Dive

### Built-in Logger

Create a `Logger` instance with the class name as context so every log line identifies its source.

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

Passing the class name (`UsersService.name`) to the constructor adds a `[UsersService]` prefix to every log entry.

### Global Logger Configuration

Filter log levels at bootstrap to control verbosity per environment.

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

Passing an array of level names enables only those levels; omitting `debug` and `verbose` keeps production logs concise.

### Custom Logger (Winston)

Implement the `LoggerService` interface to replace the built-in logger with Winston or any other library.

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

Once set with `app.useLogger()`, all NestJS internal logs (e.g., route registration) also flow through Winston.

### Request Logging

Build an interceptor that records the duration of each HTTP request for monitoring.

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

The `tap()` operator runs after the response completes, giving an accurate end-to-end duration.

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

- NestJS Logger provides contextual logging with levels: error, warn, log, debug, verbose
- Replace the built-in logger with Winston or Pino for production-grade features
- Use structured JSON logging for integration with log aggregation services
- Include correlation IDs for request tracing in distributed systems
- Never log sensitive data such as passwords, tokens, or PII

## Code Examples

**Using the built-in Logger service with context-aware log messages**

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


## Resources

- [Logger](https://docs.nestjs.com/techniques/logger) — Official logger guide

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*