---
source_course: "nestjs-techniques"
source_lesson: "nestjs-techniques-swagger-cli"
---

# Schema Generation & SDK

## Introduction

OpenAPI schemas can generate client SDKs, type definitions, and mock servers. This enables type-safe API clients and ensures frontend/backend stay in sync.

## Key Concepts

- **Schema Export**: Save OpenAPI spec as JSON/YAML
- **Code Generation**: Generate typed clients from schema
- **Schema Validation**: Validate requests/responses against spec
- **Mock Server**: Generate mock API from schema

## Real World Context

Schema generation enables:
- Type-safe frontend API clients
- Mobile app SDK generation
- Contract testing
- Documentation portals

## Deep Dive

### Export OpenAPI Schema

```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  const config = new DocumentBuilder()
    .setTitle('API')
    .setVersion('1.0')
    .build();

  const document = SwaggerModule.createDocument(app, config);

  // Export to file
  const fs = require('fs');
  fs.writeFileSync('./openapi.json', JSON.stringify(document, null, 2));

  SwaggerModule.setup('api', app, document);
  await app.listen(3000);
}
```

### Generate TypeScript Client

```bash
npm install -g openapi-generator-cli

# Generate TypeScript Axios client
openapi-generator-cli generate \
  -i openapi.json \
  -g typescript-axios \
  -o ./generated/api-client
```

### Usage of Generated Client

```typescript
import { UsersApi, Configuration } from './generated/api-client';

const config = new Configuration({
  basePath: 'http://localhost:3000',
  accessToken: 'your-jwt-token',
});

const usersApi = new UsersApi(config);

// Type-safe API calls
const users = await usersApi.usersControllerFindAll();
const user = await usersApi.usersControllerFindOne({ id: '123' });
```

### Programmatic Schema Access

```typescript
// Script to generate schema on build
import { NestFactory } from '@nestjs/core';
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';
import { AppModule } from './app.module';
import * as fs from 'fs';

async function generateSchema() {
  const app = await NestFactory.create(AppModule, { logger: false });
  await app.init();

  const config = new DocumentBuilder()
    .setTitle('API')
    .setVersion('1.0')
    .build();

  const document = SwaggerModule.createDocument(app, config);
  fs.writeFileSync('./dist/openapi.json', JSON.stringify(document, null, 2));

  await app.close();
  console.log('OpenAPI schema generated');
}

generateSchema();
```

### NPM Script Integration

```json
// package.json
{
  "scripts": {
    "build": "nest build",
    "postbuild": "ts-node scripts/generate-schema.ts",
    "generate:client": "openapi-generator-cli generate -i dist/openapi.json -g typescript-axios -o ../frontend/src/api"
  }
}
```

### Schema Validation Middleware

```typescript
import * as OpenApiValidator from 'express-openapi-validator';

app.use(
  OpenApiValidator.middleware({
    apiSpec: './openapi.json',
    validateRequests: true,
    validateResponses: true,
  }),
);
```

## Common Pitfalls

1. **Stale schema**: Regenerate on every build.
2. **Missing models**: Ensure all DTOs are included.
3. **Version mismatch**: Keep schema version in sync with API.

## Best Practices

- Generate schema as part of build process
- Version schemas alongside API versions
- Use generated clients in frontend projects
- Validate requests/responses in development
- Store schemas in version control

## Summary

OpenAPI schemas enable type-safe client generation. Export schemas during build, generate clients with openapi-generator-cli, and validate requests/responses. This ensures frontend and backend stay synchronized.

## Resources

- [OpenAPI CLI Plugin](https://docs.nestjs.com/openapi/cli-plugin) — Schema generation automation

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*