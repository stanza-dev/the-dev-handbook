---
source_course: "nestjs-essentials"
source_lesson: "nestjs-essentials-controllers"
---

# Controllers

## Introduction

Every web API needs to handle HTTP requests—that's where controllers come in. Controllers are the entry point for client requests, defining routes and orchestrating responses. Think of them as traffic directors: they receive requests, delegate work to services, and send back responses.

## Key Concepts

- **Controller**: A class decorated with `@Controller()` that handles incoming HTTP requests
- **Route Handler**: A method within a controller that responds to a specific HTTP method and path
- **Route Prefix**: The base path for all routes in a controller (e.g., `/cats`)
- **Decorators**: Metadata annotations that define routing and parameter extraction

## Real World Context

In a production API, controllers:
- Define your API's public interface (endpoints)
- Handle request validation and response formatting
- Remain thin—delegating business logic to services
- Provide clear separation between HTTP concerns and business rules

## Deep Dive

### Creating a Controller

```typescript
import { Controller, Get, Post, Body, Param, Query } from '@nestjs/common';
import { CreateCatDto } from './dto/create-cat.dto';
import { CatsService } from './cats.service';

@Controller('cats')
export class CatsController {
  constructor(private readonly catsService: CatsService) {}

  @Post()
  create(@Body() createCatDto: CreateCatDto) {
    return this.catsService.create(createCatDto);
  }

  @Get()
  findAll(@Query('limit') limit: number) {
    return this.catsService.findAll(limit);
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.catsService.findOne(id);
  }
}
```

### HTTP Method Decorators

| Decorator | HTTP Method | Typical Use |
|-----------|-------------|-------------|
| `@Get()` | GET | Retrieve resources |
| `@Post()` | POST | Create resources |
| `@Put()` | PUT | Replace resources |
| `@Patch()` | PATCH | Partial update |
| `@Delete()` | DELETE | Remove resources |

### Parameter Decorators

| Decorator | Extracts | Example |
|-----------|----------|--------|
| `@Body()` | Request body | `@Body() dto: CreateCatDto` |
| `@Param()` | Route params | `@Param('id') id: string` |
| `@Query()` | Query string | `@Query('page') page: number` |
| `@Headers()` | HTTP headers | `@Headers('authorization') token` |

### Response Handling

By default, NestJS returns:
- Status **200** for GET, PUT, PATCH, DELETE
- Status **201** for POST

Customize with `@HttpCode()` and `@Header()`:

```typescript
@Post()
@HttpCode(204)
@Header('Cache-Control', 'none')
create() { ... }
```

## Common Pitfalls

1. **Business logic in controllers**: Controllers should be thin. Extract logic to services for testability and reusability.
2. **Not using DTOs**: Raw `@Body() body: any` loses type safety. Always define DTO classes.
3. **Forgetting async/await**: Database operations are async. Forgetting `await` returns unresolved promises.

## Best Practices

- **One controller per resource**: `UsersController`, `ProductsController`, `OrdersController`
- **Use DTOs for input validation**: Combine with `ValidationPipe` for automatic validation
- **Keep handlers under 10 lines**: Delegate to services
- **Use semantic HTTP methods**: GET for reads, POST for creates, etc.

## Summary

Controllers handle HTTP requests using decorators like `@Controller()`, `@Get()`, `@Post()`, and parameter extractors like `@Body()` and `@Param()`. They should remain thin, delegating business logic to injectable services.

## Resources

- [Controllers Documentation](https://docs.nestjs.com/controllers) — Official guide to creating and configuring controllers

---

> 📘 *This lesson is part of the [NestJS Essentials](https://stanza.dev/courses/nestjs-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*