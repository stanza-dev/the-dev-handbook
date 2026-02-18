---
source_course: "nestjs-fundamentals"
source_lesson: "nestjs-fundamentals-error-logging"
---

# Error Logging & Monitoring

## Introduction

Errors in production are inevitable. Proper logging and monitoring ensure you know about errors before users report them, understand their impact, and have the context needed to fix them.

## Key Concepts

- **Structured Logging**: JSON logs with consistent fields
- **Error Context**: Information needed to debug
- **Error Tracking**: Aggregating and alerting on errors
- **Correlation ID**: Linking related log entries

## Real World Context

Proper error logging enables:
- Proactive issue detection
- Fast incident response
- Root cause analysis
- Trend identification

## Deep Dive

### Structured Error Logging

```typescript
@Injectable()
export class ErrorLoggerService {
  private readonly logger = new Logger('Error');

  logError(
    error: Error,
    context: {
      correlationId?: string;
      userId?: string;
      path?: string;
      method?: string;
      body?: any;
      [key: string]: any;
    },
  ) {
    const errorLog = {
      name: error.name,
      message: error.message,
      stack: error.stack,
      code: (error as any).code,
      ...context,
      timestamp: new Date().toISOString(),
    };

    this.logger.error(JSON.stringify(errorLog));

    // Send to error tracking service
    this.sendToErrorTracking(errorLog);
  }

  private sendToErrorTracking(errorLog: any) {
    // Integration with Sentry, Datadog, etc.
  }
}
```

### Global Exception Filter with Logging

```typescript
@Catch()
export class GlobalExceptionFilter implements ExceptionFilter {
  constructor(
    private errorLogger: ErrorLoggerService,
    private readonly httpAdapterHost: HttpAdapterHost,
  ) {}

  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const request = ctx.getRequest();
    const response = ctx.getResponse();

    const status = exception instanceof HttpException
      ? exception.getStatus()
      : HttpStatus.INTERNAL_SERVER_ERROR;

    const correlationId = request.headers['x-correlation-id'] || uuid();

    // Log with full context
    this.errorLogger.logError(exception as Error, {
      correlationId,
      userId: request.user?.id,
      path: request.url,
      method: request.method,
      body: this.sanitizeBody(request.body),
      query: request.query,
      headers: this.sanitizeHeaders(request.headers),
      statusCode: status,
    });

    // Return sanitized error to client
    response.status(status).json({
      statusCode: status,
      message: this.getClientMessage(exception, status),
      correlationId,
      timestamp: new Date().toISOString(),
    });
  }

  private sanitizeBody(body: any): any {
    const sensitiveFields = ['password', 'token', 'secret', 'creditCard'];
    return Object.fromEntries(
      Object.entries(body || {}).map(([key, value]) =>
        sensitiveFields.includes(key) ? [key, '[REDACTED]'] : [key, value]
      ),
    );
  }

  private getClientMessage(exception: unknown, status: number): string {
    if (exception instanceof HttpException) {
      return exception.message;
    }
    // Don't expose internal errors to clients
    return status >= 500 ? 'Internal server error' : 'An error occurred';
  }
}
```

### Correlation ID Middleware

```typescript
@Injectable()
export class CorrelationIdMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    const correlationId = req.headers['x-correlation-id'] || uuid();
    req.headers['x-correlation-id'] = correlationId;
    res.setHeader('x-correlation-id', correlationId);
    next();
  }
}
```

### Error Metrics

```typescript
@Injectable()
export class ErrorMetricsService {
  private errorCounts = new Map<string, number>();

  recordError(errorType: string, path: string) {
    const key = `${errorType}:${path}`;
    this.errorCounts.set(key, (this.errorCounts.get(key) || 0) + 1);
  }

  getMetrics() {
    return Object.fromEntries(this.errorCounts);
  }

  // Reset metrics periodically
  @Cron('0 * * * *') // Every hour
  resetMetrics() {
    this.errorCounts.clear();
  }
}
```

### Integration Example (Sentry)

```typescript
import * as Sentry from '@sentry/node';

@Injectable()
export class SentryService {
  constructor() {
    Sentry.init({
      dsn: process.env.SENTRY_DSN,
      environment: process.env.NODE_ENV,
    });
  }

  captureException(error: Error, context?: Record<string, any>) {
    Sentry.withScope((scope) => {
      if (context) {
        Object.entries(context).forEach(([key, value]) => {
          scope.setExtra(key, value);
        });
      }
      Sentry.captureException(error);
    });
  }
}
```

## Common Pitfalls

1. **Logging sensitive data**: Passwords, tokens, PII in logs.
2. **Too much logging**: Noise obscures real issues.
3. **No correlation**: Can't follow a request across services.

## Best Practices

- Use structured JSON logging
- Include correlation IDs in all logs
- Sanitize sensitive data before logging
- Aggregate errors by type and frequency
- Set up alerts for error rate spikes
- Log context needed for debugging

## Summary

Effective error logging uses structured JSON with consistent fields. Include correlation IDs, sanitize sensitive data, and integrate with error tracking services. Monitor error rates and set up alerts for anomalies.

## Resources

- [Logger](https://docs.nestjs.com/techniques/logger) — Logging best practices

---

> 📘 *This lesson is part of the [NestJS Fundamentals](https://stanza.dev/courses/nestjs-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*