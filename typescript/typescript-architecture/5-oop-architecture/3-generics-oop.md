---
source_course: "typescript-architecture"
source_lesson: "typescript-architecture-generics-oop"
---

# Generic Repositories & Services

## Introduction
Generic base classes let you write database access and business logic once and reuse it across every entity in your application. A `BaseRepository<T>` provides CRUD operations for any entity type, while a `BaseService<T>` adds business rules on top.

## Key Concepts
- **Generic Repository**: A base class parameterized by entity type that provides standard CRUD operations.
- **Generic Constraint**: Using `extends` to require the entity type has certain properties (e.g., an `id` field).
- **Template Method**: An abstract method in the base class that subclasses override for entity-specific logic.

## Real World Context
Every ORM (Prisma, TypeORM, Drizzle) and most backend frameworks use generic repositories internally. Writing your own teaches you the pattern and lets you customize behavior that ORMs do not support out of the box.

## Deep Dive
### Generic Repository

```typescript
interface HasId {
    id: string;
}

class BaseRepository<T extends HasId> {
    protected items = new Map<string, T>();

    async findById(id: string): Promise<T | undefined> {
        return this.items.get(id);
    }

    async findAll(): Promise<T[]> {
        return Array.from(this.items.values());
    }

    async create(entity: T): Promise<T> {
        this.items.set(entity.id, entity);
        return entity;
    }

    async update(id: string, data: Partial<T>): Promise<T | undefined> {
        const existing = this.items.get(id);
        if (!existing) return undefined;
        const updated = { ...existing, ...data };
        this.items.set(id, updated);
        return updated;
    }

    async delete(id: string): Promise<boolean> {
        return this.items.delete(id);
    }
}
```

### Entity-Specific Repository

```typescript
interface User extends HasId {
    name: string;
    email: string;
    role: "admin" | "user";
}

class UserRepository extends BaseRepository<User> {
    async findByEmail(email: string): Promise<User | undefined> {
        const users = await this.findAll();
        return users.find(u => u.email === email);
    }

    async findAdmins(): Promise<User[]> {
        const users = await this.findAll();
        return users.filter(u => u.role === "admin");
    }
}
```

`UserRepository` inherits all CRUD methods from `BaseRepository<User>` and adds user-specific queries.

### Generic Service Layer

```typescript
class BaseService<T extends HasId, R extends BaseRepository<T>> {
    constructor(protected repository: R) {}

    async getById(id: string): Promise<T> {
        const entity = await this.repository.findById(id);
        if (!entity) throw new Error(`Entity ${id} not found`);
        return entity;
    }

    async getAll(): Promise<T[]> {
        return this.repository.findAll();
    }
}

class UserService extends BaseService<User, UserRepository> {
    async getAdminEmails(): Promise<string[]> {
        const admins = await this.repository.findAdmins();
        return admins.map(a => a.email);
    }
}
```

## Common Pitfalls
1. **Leaky abstractions** — A generic repository cannot handle every query pattern. Do not force complex joins or aggregations into the base class.
2. **Over-generalization** — If entities have very different access patterns, separate repositories may be clearer than forcing everything through a base class.

## Best Practices
1. **Require an `id` constraint** — Use `T extends HasId` so the base class can implement `findById` and `delete` generically.
2. **Keep base classes thin** — Only put truly universal operations (CRUD) in the base. Push entity-specific logic into subclasses.

## Summary
- Generic repositories provide reusable CRUD for any entity type.
- Constraints like `extends HasId` ensure the base class can access required properties.
- Entity-specific repositories extend the base with custom queries.
- Keep base classes thin to avoid leaky abstractions.

## Code Examples

**A generic Repository class that provides typed CRUD for any entity — instantiate with a specific type to get full type safety**

```typescript
interface HasId { id: string; }

// Generic CRUD repository — works for any entity with an id
class Repository<T extends HasId> {
    private store = new Map<string, T>();

    save(entity: T): T {
        this.store.set(entity.id, entity);
        return entity;
    }

    find(id: string): T | undefined {
        return this.store.get(id);
    }

    all(): T[] {
        return [...this.store.values()];
    }
}

// Typed repositories — no extra code needed for basic CRUD
interface Product extends HasId { name: string; price: number; }
const products = new Repository<Product>();

products.save({ id: "p-1", name: "Widget", price: 9.99 });
const widget = products.find("p-1"); // Product | undefined
```


## Resources

- [TypeScript Handbook: Generics — Generic Classes](https://www.typescriptlang.org/docs/handbook/2/generics.html#generic-classes) — Official docs on creating and using generic classes in TypeScript

---

> 📘 *This lesson is part of the [TypeScript Architecture & Patterns](https://stanza.dev/courses/typescript-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*