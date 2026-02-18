---
source_course: "nestjs-performance"
source_lesson: "nestjs-performance-stateless-design"
---

# Stateless Design

## Introduction

Stateless applications don't store session data locally. Each request contains all information needed to process it. This enables horizontal scaling—any server can handle any request.

## Key Concepts

- **Stateless**: No server-side session storage
- **JWT**: Self-contained tokens for authentication
- **External State**: Store state in database/Redis, not memory
- **Horizontal Scaling**: Adding more instances

## Real World Context

Stateless design enables:
- Load balancing across multiple instances
- Zero-downtime deployments
- Auto-scaling based on demand
- Container orchestration (Kubernetes)

## Deep Dive

### Stateless Authentication

```typescript
// Stateful (bad for scaling)
// Session stored in server memory
request.session.userId = user.id;

// Stateless (good for scaling)
// Token contains all needed information
const token = jwt.sign(
  { sub: user.id, email: user.email, roles: user.roles },
  secret,
  { expiresIn: '1h' },
);
```

### External Session Store

```typescript
import * as session from 'express-session';
import * as RedisStore from 'connect-redis';
import { createClient } from 'redis';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  const redisClient = createClient({ url: process.env.REDIS_URL });
  await redisClient.connect();

  app.use(
    session({
      store: new RedisStore({ client: redisClient }),
      secret: process.env.SESSION_SECRET,
      resave: false,
      saveUninitialized: false,
    }),
  );

  await app.listen(3000);
}
```

### Avoiding Singleton State

```typescript
// BAD: In-memory state
@Injectable()
export class StatefulService {
  private cache = new Map<string, any>();  // Lost on restart
  private counter = 0;  // Different per instance
}

// GOOD: External state
@Injectable()
export class StatelessService {
  constructor(
    @Inject(CACHE_MANAGER) private cache: Cache,  // Redis
    @InjectRepository(Counter) private counterRepo: Repository<Counter>,
  ) {}

  async incrementCounter(name: string): Promise<number> {
    const counter = await this.counterRepo.findOne({ where: { name } });
    counter.value++;
    await this.counterRepo.save(counter);
    return counter.value;
  }
}
```

### Request-Scoped State

```typescript
// Request context for request-specific data
@Injectable({ scope: Scope.REQUEST })
export class RequestContextService {
  private correlationId: string;
  private userId: string;

  setContext(correlationId: string, userId: string) {
    this.correlationId = correlationId;
    this.userId = userId;
  }

  getCorrelationId(): string {
    return this.correlationId;
  }
}
```

### Health Checks for Load Balancers

```typescript
import { HealthCheckService, HttpHealthIndicator, TypeOrmHealthIndicator } from '@nestjs/terminus';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private http: HttpHealthIndicator,
    private db: TypeOrmHealthIndicator,
  ) {}

  @Get()
  check() {
    return this.health.check([
      () => this.db.pingCheck('database'),
    ]);
  }

  @Get('ready')
  readiness() {
    return this.health.check([
      () => this.db.pingCheck('database'),
      () => this.http.pingCheck('external-api', 'https://api.example.com'),
    ]);
  }

  @Get('live')
  liveness() {
    return { status: 'ok' };
  }
}
```

## Common Pitfalls

1. **Local file storage**: Use S3/GCS instead.
2. **In-memory caching**: Use Redis for shared cache.
3. **Server-side sessions**: Use JWTs or external session store.

## Best Practices

- Store all state in external services
- Use JWTs for stateless authentication
- Use Redis for caching and sessions
- Implement proper health checks
- Design for instance failure

## Summary

Stateless design stores no state in the application instance. Use JWTs for auth, Redis for caching/sessions, and databases for persistence. This enables horizontal scaling and zero-downtime deployments.

## Resources

- [Health Checks](https://docs.nestjs.com/recipes/terminus) — Health checks for scaling

---

> 📘 *This lesson is part of the [NestJS High Performance](https://stanza.dev/courses/nestjs-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*