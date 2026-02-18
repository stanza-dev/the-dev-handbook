---
source_course: "nestjs-essentials"
source_lesson: "nestjs-essentials-installation-and-setup"
---

# Installation & CLI

## Introduction

Every great application starts with a solid foundation. The Nest CLI is your primary tool for scaffolding projects, generating components, and maintaining consistency across your codebase. Understanding the CLI workflow will save you hours of boilerplate writing.

## Key Concepts

- **Nest CLI**: A command-line tool for generating and managing NestJS projects
- **Scaffold**: Auto-generated project structure with configuration files
- **Bootstrap**: The process of initializing and starting the application
- **NestFactory**: The core class that creates application instances

## Real World Context

In production teams, consistency is paramount. The CLI ensures every developer generates modules, services, and controllers with the same structure. This eliminates debates about file naming, folder structure, and boilerplate patterns.

## Deep Dive

### Installing the CLI

```bash
npm i -g @nestjs/cli
```

### Creating a New Project

```bash
nest new my-api
```

The CLI prompts you to choose a package manager (npm, yarn, or pnpm) and scaffolds:

```
my-api/
├── src/
│   ├── app.controller.ts
│   ├── app.module.ts
│   ├── app.service.ts
│   └── main.ts
├── test/
├── nest-cli.json
├── package.json
└── tsconfig.json
```

### Bootstrapping the Application

The entry point is `main.ts`:

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

### Essential CLI Commands

| Command | Description |
|---------|------------|
| `nest new <name>` | Create a new project |
| `nest generate module <name>` | Generate a module |
| `nest generate controller <name>` | Generate a controller |
| `nest generate service <name>` | Generate a service |
| `nest build` | Compile the application |
| `nest start --watch` | Start in development mode with hot reload |

## Common Pitfalls

1. **Not using `--watch` in development**: Running `nest start` without `--watch` means you restart manually on every change. Always use `nest start --watch` or `npm run start:dev`.
2. **Installing CLI locally only**: Install globally (`npm i -g`) for easy access to `nest` commands anywhere. Local installation works but requires `npx nest` prefix.
3. **Ignoring the `nest-cli.json` configuration**: This file controls compilation, asset copying, and monorepo settings. Misconfiguring it breaks builds.

## Best Practices

- **Use the shorthand**: `nest g mo users` instead of `nest generate module users`
- **Generate related files together**: `nest g resource users` creates module, controller, service, and DTOs
- **Keep `main.ts` minimal**: Configuration belongs in modules, not the bootstrap file
- **Use environment variables for port**: `process.env.PORT ?? 3000` enables cloud deployment

## Summary

The Nest CLI accelerates development by scaffolding projects and generating components with consistent structure. The `NestFactory.create()` method bootstraps your application by building the dependency graph from `AppModule`. Master the CLI commands to maintain velocity as your project grows.

## Resources

- [Nest CLI Overview](https://docs.nestjs.com/cli/overview) — Complete guide to Nest CLI commands and options
- [First Steps](https://docs.nestjs.com/first-steps) — Bootstrapping your first NestJS application

---

> 📘 *This lesson is part of the [NestJS Essentials](https://stanza.dev/courses/nestjs-essentials) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*