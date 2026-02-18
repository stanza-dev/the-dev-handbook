---
source_course: "nestjs-techniques"
source_lesson: "nestjs-techniques-configuration"
---

# Configuration

## Introduction

Applications run in different environments—development, staging, production—each with different settings. Hardcoding configuration is a security risk and deployment nightmare. NestJS's ConfigModule provides type-safe, environment-aware configuration management.

## Key Concepts

- **ConfigModule**: Module for loading environment variables
- **ConfigService**: Injectable service for accessing configuration
- **.env files**: Files storing environment-specific variables
- **Configuration namespaces**: Organized, typed configuration objects

## Real World Context

Production apps need:
- Database credentials per environment
- API keys and secrets
- Feature flags
- Service URLs

The ConfigModule loads these from .env files and environment variables, with validation to catch missing values at startup.

## Deep Dive

### Basic Setup

```bash
npm install @nestjs/config
```

```typescript
import { ConfigModule } from '@nestjs/config';

@Module({
  imports: [ConfigModule.forRoot()],
})
export class AppModule {}
```

### Using ConfigService

```typescript
@Injectable()
export class DatabaseService {
  constructor(private configService: ConfigService) {}

  connect() {
    const host = this.configService.get<string>('DATABASE_HOST');
    const port = this.configService.get<number>('DATABASE_PORT', 5432);
    // ...
  }
}
```

### Custom Configuration Files

```typescript
// config/database.config.ts
export default () => ({
  database: {
    host: process.env.DATABASE_HOST,
    port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
    name: process.env.DATABASE_NAME,
  },
});

// app.module.ts
ConfigModule.forRoot({
  load: [databaseConfig],
})
```

### Schema Validation with Joi

```typescript
import * as Joi from 'joi';

ConfigModule.forRoot({
  validationSchema: Joi.object({
    NODE_ENV: Joi.string().valid('development', 'production', 'test').required(),
    PORT: Joi.number().default(3000),
    DATABASE_URL: Joi.string().required(),
  }),
})
```

## Common Pitfalls

1. **Missing .env in production**: Use actual environment variables, not .env files in production.
2. **Not validating**: Missing required config crashes the app at runtime instead of startup.
3. **Exposing secrets in logs**: Never log configuration values.

## Best Practices

- Validate configuration schema at startup
- Use typed configuration namespaces
- Never commit .env files (use .env.example)
- Use different .env files per environment (.env.development, .env.production)

## Summary

ConfigModule loads environment variables from .env files and process.env. Use ConfigService to access values with type safety. Validate configuration with Joi to catch errors at startup.

## Resources

- [Configuration](https://docs.nestjs.com/techniques/configuration) — Official configuration guide

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*