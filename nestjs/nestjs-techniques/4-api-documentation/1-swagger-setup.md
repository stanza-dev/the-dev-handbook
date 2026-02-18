---
source_course: "nestjs-techniques"
source_lesson: "nestjs-techniques-swagger-setup"
---

# OpenAPI/Swagger Setup

## Introduction

API documentation is essential for developers consuming your API. OpenAPI (formerly Swagger) provides a standard format for describing REST APIs. NestJS integrates seamlessly with Swagger to auto-generate documentation from your code.

## Key Concepts

- **OpenAPI**: Specification for describing REST APIs
- **Swagger UI**: Interactive documentation interface
- **Decorators**: NestJS decorators that add OpenAPI metadata
- **Schema Generation**: Automatic DTO-to-schema conversion

## Real World Context

Swagger documentation enables:
- Developer onboarding without reading code
- API testing directly in the browser
- Client SDK generation
- Contract-first API development

## Deep Dive

### Installation & Setup

```bash
npm install @nestjs/swagger
```

```typescript
import { NestFactory } from '@nestjs/core';
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  const config = new DocumentBuilder()
    .setTitle('My API')
    .setDescription('The API documentation')
    .setVersion('1.0')
    .addBearerAuth()
    .addTag('users')
    .addTag('products')
    .build();

  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('api', app, document);

  await app.listen(3000);
}
bootstrap();
```

### Controller Documentation

```typescript
import { ApiTags, ApiOperation, ApiResponse, ApiBearerAuth } from '@nestjs/swagger';

@ApiTags('users')
@Controller('users')
export class UsersController {
  @Get()
  @ApiOperation({ summary: 'Get all users' })
  @ApiResponse({ status: 200, description: 'Returns all users', type: [UserDto] })
  findAll(): Promise<UserDto[]> {
    return this.usersService.findAll();
  }

  @Get(':id')
  @ApiOperation({ summary: 'Get user by ID' })
  @ApiResponse({ status: 200, description: 'Returns the user', type: UserDto })
  @ApiResponse({ status: 404, description: 'User not found' })
  findOne(@Param('id') id: string): Promise<UserDto> {
    return this.usersService.findOne(id);
  }

  @Post()
  @ApiBearerAuth()
  @ApiOperation({ summary: 'Create a new user' })
  @ApiResponse({ status: 201, description: 'User created', type: UserDto })
  @ApiResponse({ status: 400, description: 'Invalid input' })
  create(@Body() createUserDto: CreateUserDto): Promise<UserDto> {
    return this.usersService.create(createUserDto);
  }
}
```

### DTO Documentation

```typescript
import { ApiProperty, ApiPropertyOptional } from '@nestjs/swagger';

export class CreateUserDto {
  @ApiProperty({
    description: 'User email address',
    example: 'user@example.com',
  })
  @IsEmail()
  email: string;

  @ApiProperty({
    description: 'User full name',
    minLength: 2,
    maxLength: 100,
    example: 'John Doe',
  })
  @IsString()
  @MinLength(2)
  name: string;

  @ApiPropertyOptional({
    description: 'User age',
    minimum: 0,
    maximum: 120,
    example: 25,
  })
  @IsOptional()
  @IsInt()
  age?: number;
}
```

### CLI Plugin for Auto-Generation

```json
// nest-cli.json
{
  "compilerOptions": {
    "plugins": [
      {
        "name": "@nestjs/swagger",
        "options": {
          "classValidatorShim": true,
          "introspectComments": true
        }
      }
    ]
  }
}
```

With the plugin, decorators are optional—metadata is inferred:

```typescript
export class CreateUserDto {
  /** User email address */
  @IsEmail()
  email: string;  // ApiProperty auto-generated

  /** User full name */
  @IsString()
  name: string;
}
```

## Common Pitfalls

1. **Missing return types**: Without types, Swagger can't generate schemas.
2. **Forgetting ApiProperty on nested objects**: Nested DTOs need decorators too.
3. **Not documenting errors**: Document 400, 401, 404 responses.

## Best Practices

- Use the CLI plugin to reduce decorator boilerplate
- Document all response types including errors
- Add examples to ApiProperty for clarity
- Group related endpoints with ApiTags
- Keep documentation in sync with code

## Summary

@nestjs/swagger integrates OpenAPI documentation into NestJS. Use decorators like @ApiOperation, @ApiResponse, and @ApiProperty to document endpoints. The CLI plugin can auto-generate much of the documentation from types.

## Resources

- [OpenAPI (Swagger)](https://docs.nestjs.com/openapi/introduction) — Official Swagger integration guide

---

> 📘 *This lesson is part of the [NestJS Techniques](https://stanza.dev/courses/nestjs-techniques) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*