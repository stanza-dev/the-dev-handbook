---
source_course: "typescript-architecture"
source_lesson: "typescript-architecture-interface-segregation"
---

# Interface Segregation Principle

## Introduction
The Interface Segregation Principle (ISP) states that no class should be forced to implement methods it does not use. In TypeScript, this means splitting large interfaces into smaller, focused ones so that implementors only depend on the methods they actually need.

## Key Concepts
- **ISP (Interface Segregation Principle)**: Clients should not be forced to depend on interfaces they do not use.
- **Fat Interface**: An interface with too many methods, forcing implementors to provide stubs for irrelevant functionality.
- **Role Interface**: A small, focused interface representing a single capability or role.

## Real World Context
A `Repository` interface with `find`, `create`, `update`, `delete`, `bulkInsert`, `aggregate`, and `stream` methods forces read-only services to implement write methods they never use. Splitting into `Readable`, `Writable`, and `Streamable` interfaces lets each consumer depend only on what it needs.

## Deep Dive
### The Problem: Fat Interfaces

```typescript
// Bad: Fat interface forces unnecessary implementations
interface UserRepository {
    findById(id: string): Promise<User>;
    findAll(): Promise<User[]>;
    create(user: User): Promise<void>;
    update(id: string, data: Partial<User>): Promise<void>;
    delete(id: string): Promise<void>;
    bulkImport(users: User[]): Promise<void>;
}

// A read-only analytics service must implement create, update, delete, bulkImport
// even though it never uses them
class AnalyticsService implements UserRepository {
    async findById(id: string) { /* real impl */ }
    async findAll() { /* real impl */ }
    async create() { throw new Error("Not supported"); } // Wasted code
    async update() { throw new Error("Not supported"); }
    async delete() { throw new Error("Not supported"); }
    async bulkImport() { throw new Error("Not supported"); }
}
```

### The Fix: Segregated Interfaces

```typescript
interface ReadableRepo<T> {
    findById(id: string): Promise<T>;
    findAll(): Promise<T[]>;
}

interface WritableRepo<T> {
    create(entity: T): Promise<void>;
    update(id: string, data: Partial<T>): Promise<void>;
    delete(id: string): Promise<void>;
}

interface BulkImportable<T> {
    bulkImport(entities: T[]): Promise<void>;
}

// Full repository composes all interfaces
interface UserRepository extends ReadableRepo<User>, WritableRepo<User>, BulkImportable<User> {}

// Analytics only depends on ReadableRepo
class AnalyticsService {
    constructor(private repo: ReadableRepo<User>) {}

    async getActiveUserCount(): Promise<number> {
        const users = await this.repo.findAll();
        return users.filter(u => u.isActive).length;
    }
}
```

Now `AnalyticsService` only depends on `ReadableRepo<User>` — it does not know or care about write methods.

### Intersection for Ad-Hoc Composition

When a function needs a specific combination:

```typescript
function migrateUsers(
    source: ReadableRepo<User>,
    target: WritableRepo<User> & BulkImportable<User>
): Promise<void> {
    // target must support both write and bulk import
}
```

## Common Pitfalls
1. **Over-segregation** — Splitting every method into its own interface creates too many tiny interfaces that are hard to discover. Group methods by cohesive capability.
2. **Breaking existing consumers** — When refactoring a fat interface, keep the original as a composed type (`interface Full extends A, B, C {}`) so existing code still compiles.

## Best Practices
1. **Group by capability** — `Readable`, `Writable`, `Streamable` are good groupings. `FindById`, `FindAll` as separate interfaces is too granular.
2. **Use generics** — Make segregated interfaces generic (`ReadableRepo<T>`) so they apply across multiple entities.

## Summary
- ISP: no class should depend on methods it does not use.
- Split fat interfaces into small role interfaces grouped by capability.
- Compose role interfaces via `extends` or intersection (`&`) for full implementations.
- Use generics to make segregated interfaces reusable across entities.

## Code Examples

**Segregated notification interfaces — EmailService implements all three while SmsService only needs Sendable**

```typescript
// Segregated interfaces for a notification system
interface Sendable {
    send(to: string, message: string): Promise<void>;
}

interface Schedulable {
    schedule(to: string, message: string, at: Date): Promise<void>;
}

interface Trackable {
    getDeliveryStatus(id: string): Promise<"sent" | "delivered" | "failed">;
}

// Email supports all three
class EmailService implements Sendable, Schedulable, Trackable {
    async send(to: string, message: string) { /* ... */ }
    async schedule(to: string, message: string, at: Date) { /* ... */ }
    async getDeliveryStatus(id: string) { return "delivered" as const; }
}

// SMS only supports sending — no scheduling or tracking
class SmsService implements Sendable {
    async send(to: string, message: string) { /* ... */ }
}
```


## Resources

- [SOLID Principles in TypeScript](https://www.typescriptlang.org/docs/handbook/2/objects.html#extending-types) — TypeScript handbook section on extending and composing interface types

---

> 📘 *This lesson is part of the [TypeScript Architecture & Patterns](https://stanza.dev/courses/typescript-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*