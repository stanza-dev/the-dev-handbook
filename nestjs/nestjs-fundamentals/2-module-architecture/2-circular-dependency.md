---
source_course: "nestjs-fundamentals"
source_lesson: "nestjs-fundamentals-circular-dependency"
---

# Circular Dependencies

## Introduction

Circular dependencies occur when Module A depends on Module B, and B depends on A. This creates an initialization problem—neither can fully initialize without the other. NestJS provides forwardRef() to break these cycles.

## Key Concepts

- **Circular Dependency**: A → B → A (module or provider level)
- **forwardRef()**: Lazy reference that resolves after initialization
- **Module Ref**: Alternative for accessing services post-initialization

## Real World Context

Consider Users and Organizations modules. Users belong to organizations, organizations have owner users. Each service might need the other, creating a circular dependency.

## Deep Dive

### Module-Level Circular Dependency

```typescript
// users.module.ts
@Module({
  imports: [forwardRef(() => OrganizationsModule)],
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}

// organizations.module.ts
@Module({
  imports: [forwardRef(() => UsersModule)],
  providers: [OrganizationsService],
  exports: [OrganizationsService],
})
export class OrganizationsModule {}
```

### Provider-Level Circular Dependency

```typescript
// users.service.ts
@Injectable()
export class UsersService {
  constructor(
    @Inject(forwardRef(() => OrganizationsService))
    private orgsService: OrganizationsService,
  ) {}
}

// organizations.service.ts
@Injectable()
export class OrganizationsService {
  constructor(
    @Inject(forwardRef(() => UsersService))
    private usersService: UsersService,
  ) {}
}
```

### Using ModuleRef (Alternative)

```typescript
@Injectable()
export class UsersService implements OnModuleInit {
  private orgsService: OrganizationsService;

  constructor(private moduleRef: ModuleRef) {}

  onModuleInit() {
    this.orgsService = this.moduleRef.get(OrganizationsService, { strict: false });
  }
}
```

### Detecting Circular Dependencies

NestJS throws a clear error:
```
Nest cannot create the ... instance.
A potential circular dependency has been detected.
```

## Common Pitfalls

1. **forwardRef on only one side**: You need forwardRef on BOTH sides of the circular dependency.
2. **Architectural smell**: Circular dependencies often indicate design issues. Consider refactoring.
3. **Performance impact**: Forward references add initialization complexity.

## Best Practices

- First, try to refactor to avoid circular dependencies
- Extract shared logic to a third module both can import
- Use events/messages for decoupled communication
- Document why forwardRef was necessary

## Summary

forwardRef() breaks circular dependencies by lazily resolving references. Apply it to both sides of the cycle. Better yet, refactor your architecture to avoid circular dependencies—they often indicate a design problem.

## Resources

- [Circular Dependency](https://docs.nestjs.com/fundamentals/circular-dependency) — Official circular dependency guide

---

> 📘 *This lesson is part of the [NestJS Fundamentals](https://stanza.dev/courses/nestjs-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*