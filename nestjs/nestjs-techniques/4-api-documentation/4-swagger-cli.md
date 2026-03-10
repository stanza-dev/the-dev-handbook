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

Write the generated OpenAPI document to a JSON file during bootstrap for downstream tooling.

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

The exported `openapi.json` can be consumed by code generators, linters, and CI validation tools.

### Generate TypeScript Client

Use `openapi-generator-cli` to produce a fully typed API client from the exported schema.

```bash
npm install -g openapi-generator-cli

# Generate TypeScript Axios client
openapi-generator-cli generate \
  -i openapi.json \
  -g typescript-axios \
  -o ./generated/api-client
```

The `-g typescript-axios` flag selects the Axios-based TypeScript generator; other options include `typescript-fetch` and `typescript-node`.

### Usage of Generated Client

Import the generated API class and configuration to make type-safe requests with full autocomplete.

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

Method names follow the pattern `controllerNameMethodName`, derived from your NestJS controller and method names.

### Programmatic Schema Access

Create a standalone script that boots the app, extracts the schema, and shuts down without starting the HTTP server.

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

Using `app.init()` instead of `app.listen()` loads all modules without binding a port, which is faster for CI.

### NPM Script Integration

Automate schema generation and client code generation as part of your build pipeline.

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

The `postbuild` hook ensures the schema is always regenerated after compilation.

### Schema Validation Middleware

Add request and response validation against the OpenAPI spec to catch contract violations in development.

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

Enabling `validateResponses` catches cases where your code returns fields not documented in the schema.

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

## Code Examples

**Exporting the OpenAPI schema to a JSON file during application bootstrap**

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


## Resources

- [OpenAPI CLI Plugin](https://docs.nestjs.com/openapi/cli-plugin) — Schema generation automation

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*