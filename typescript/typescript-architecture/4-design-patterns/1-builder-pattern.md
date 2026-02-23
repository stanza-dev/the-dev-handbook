---
source_course: "typescript-architecture"
source_lesson: "typescript-architecture-builder-pattern"
---

# Type-Safe Builder Pattern

## Introduction
The Builder pattern constructs complex objects step-by-step, separating construction from representation. In TypeScript, we can make builders that enforce required fields at compile time — calling `.build()` before setting all required properties produces a type error, not a runtime error.

## Key Concepts
- **Builder Pattern**: An object that accumulates configuration via method calls and produces a final product via a `build()` method.
- **Fluent Interface**: Method chaining via `return this`, allowing `builder.setA().setB().build()`.
- **Phantom Type State**: Using generic type parameters to track which properties have been set, enabling compile-time enforcement of required fields.

## Real World Context
HTTP request builders, query builders (like Knex), and configuration objects (like Webpack configs) all use the Builder pattern. TypeScript's type system can ensure you never forget a required field — no more "missing URL" runtime errors.

## Deep Dive
### Basic Builder with Method Chaining

```typescript
class RequestBuilder {
    private url = "";
    private method: "GET" | "POST" = "GET";
    private headers: Record<string, string> = {};

    setUrl(url: string): this {
        this.url = url;
        return this;
    }

    setMethod(method: "GET" | "POST"): this {
        this.method = method;
        return this;
    }

    addHeader(key: string, value: string): this {
        this.headers[key] = value;
        return this;
    }

    build() {
        if (!this.url) throw new Error("URL is required");
        return { url: this.url, method: this.method, headers: this.headers };
    }
}

const request = new RequestBuilder()
    .setUrl("https://api.example.com")
    .setMethod("POST")
    .addHeader("Content-Type", "application/json")
    .build();
```

Using `this` as the return type allows subclasses to chain methods without losing their specific type.

### Type-State Builder (Compile-Time Safety)

Use a generic parameter to track which fields have been set:

```typescript
type BuilderState = { url: boolean; method: boolean };

class TypedBuilder<State extends BuilderState = { url: false; method: false }> {
    private data: Partial<{ url: string; method: string }> = {};

    setUrl(url: string): TypedBuilder<State & { url: true }> {
        this.data.url = url;
        return this as any;
    }

    setMethod(method: string): TypedBuilder<State & { method: true }> {
        this.data.method = method;
        return this as any;
    }

    build(this: TypedBuilder<{ url: true; method: true }>): { url: string; method: string } {
        return this.data as { url: string; method: string };
    }
}

new TypedBuilder().setUrl("/api").setMethod("GET").build(); // OK
// new TypedBuilder().setUrl("/api").build(); // Error: method not set
```

## Common Pitfalls
1. **Runtime validation in build()** — The basic builder throws at runtime if a field is missing. Prefer the type-state pattern to catch this at compile time.
2. **Forgetting `return this`** — Every setter must return `this` for chaining to work. A void return breaks the fluent interface.

## Best Practices
1. **Use the basic builder for simple cases** — Type-state builders add complexity. Use them only when the number of required fields justifies the overhead.
2. **Make builders immutable** — Each method returns a new builder instance instead of mutating `this`, preventing shared-state bugs.

## Summary
- The Builder pattern constructs complex objects step-by-step with method chaining.
- Returning `this` enables fluent interfaces that work with subclasses.
- Type-state builders use generic parameters to enforce required fields at compile time.
- Prefer immutable builders for thread-safe, predictable construction.

## Code Examples

**A fluent SQL query builder — method chaining via 'return this' enables a readable, step-by-step construction API**

```typescript
class QueryBuilder {
    private table = "";
    private conditions: string[] = [];
    private limit?: number;

    from(table: string): this {
        this.table = table;
        return this;
    }

    where(condition: string): this {
        this.conditions.push(condition);
        return this;
    }

    take(n: number): this {
        this.limit = n;
        return this;
    }

    toSQL(): string {
        let sql = `SELECT * FROM ${this.table}`;
        if (this.conditions.length) sql += ` WHERE ${this.conditions.join(" AND ")}`;
        if (this.limit) sql += ` LIMIT ${this.limit}`;
        return sql;
    }
}

const query = new QueryBuilder()
    .from("users")
    .where("age > 18")
    .where("active = true")
    .take(10)
    .toSQL();
// "SELECT * FROM users WHERE age > 18 AND active = true LIMIT 10"
```


## Resources

- [TypeScript Handbook: Classes](https://www.typescriptlang.org/docs/handbook/2/classes.html) — Official docs on TypeScript classes, including method return types and 'this' types

---

> 📘 *This lesson is part of the [TypeScript Architecture & Patterns](https://stanza.dev/courses/typescript-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*