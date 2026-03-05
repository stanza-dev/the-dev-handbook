---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-builder"
---

# The Builder Pattern

## Introduction

The Builder pattern constructs complex objects step by step. Instead of a constructor with many parameters, you chain method calls to configure the object. This makes code readable and prevents errors from parameter ordering.

## Key Concepts

**Builder**: Object that accumulates configuration through method chaining.

**Fluent Interface**: Methods return `this` for chaining.

**Director**: Optional class that defines build steps order.

## Real World Context

Query builders (Knex), test data builders, configuration objects, HTTP request builders—builders appear throughout JavaScript libraries.

## Deep Dive

### Basic Builder

This builder accumulates user properties step by step, with each setter returning `this` to enable fluent chaining:

```javascript
class UserBuilder {
  #user = {};
  
  setName(name) {
    this.#user.name = name;
    return this;  // Enable chaining
  }
  
  setEmail(email) {
    this.#user.email = email;
    return this;
  }
  
  setRole(role) {
    this.#user.role = role;
    return this;
  }
  
  setPermissions(...permissions) {
    this.#user.permissions = permissions;
    return this;
  }
  
  build() {
    // Validation
    if (!this.#user.name || !this.#user.email) {
      throw new Error('Name and email required');
    }
    return { ...this.#user };
  }
}

const user = new UserBuilder()
  .setName('Alice')
  .setEmail('alice@example.com')
  .setRole('admin')
  .setPermissions('read', 'write')
  .build();
```

The `build()` method validates required fields and returns a frozen copy of the accumulated data, ensuring the builder cannot produce an invalid object.

### Query Builder Example

Query builders are one of the most common real-world uses of this pattern. Each method appends a clause to the SQL query being constructed:

```javascript
class QueryBuilder {
  #query = { select: '*', from: '', where: [], orderBy: null, limit: null };
  
  select(...fields) {
    this.#query.select = fields.length ? fields.join(', ') : '*';
    return this;
  }
  
  from(table) {
    this.#query.from = table;
    return this;
  }
  
  where(condition) {
    this.#query.where.push(condition);
    return this;
  }
  
  orderBy(field, direction = 'ASC') {
    this.#query.orderBy = `${field} ${direction}`;
    return this;
  }
  
  limit(n) {
    this.#query.limit = n;
    return this;
  }
  
  build() {
    let sql = `SELECT ${this.#query.select} FROM ${this.#query.from}`;
    if (this.#query.where.length) {
      sql += ` WHERE ${this.#query.where.join(' AND ')}`;
    }
    if (this.#query.orderBy) sql += ` ORDER BY ${this.#query.orderBy}`;
    if (this.#query.limit) sql += ` LIMIT ${this.#query.limit}`;
    return sql;
  }
}

const query = new QueryBuilder()
  .select('id', 'name', 'email')
  .from('users')
  .where('active = true')
  .where('age > 18')
  .orderBy('name')
  .limit(10)
  .build();
```

Notice how multiple `.where()` calls accumulate conditions joined by `AND`. The builder makes complex query construction readable and safe.

### Functional Builder

You can also build without classes using closures. Note: use regular methods (shorthand syntax) so that `this` refers to the returned object, enabling chaining.

```javascript
const createRequest = () => {
  const config = { method: 'GET', headers: {} };
  
  const builder = {
    method(m) { config.method = m; return builder; },
    header(k, v) { config.headers[k] = v; return builder; },
    body(b) { config.body = b; return builder; },
    build() { return { ...config }; }
  };
  return builder;
};
```

Each method returns the `builder` object explicitly rather than `this`, since arrow functions do not have their own `this` binding. By returning the `builder` reference explicitly, we avoid the `this` pitfall of arrow functions (which don't have their own `this`).

## Common Pitfalls

1. **Mutable state issues**: Reset or clone between builds.
2. **Too many methods**: Builder becomes unwieldy.
3. **Missing validation**: build() should validate.

## Best Practices

- **Always return this**: For fluent chaining.
- **Validate in build()**: Catch errors early.
- **Reset state after build**: Or create new builder per object.
- **Use for 3+ parameters**: Simpler cases don't need builders.

## Summary

Builders construct complex objects step by step through method chaining. Always return `this` for fluent interface. Validate in build(). Use for objects with many configuration options.

## Code Examples

**Builder pattern for SQL queries — each method returns `this` for fluent chaining, build() produces the final result**

```javascript
class QueryBuilder {
  #table = '';
  #conditions = [];
  #orderBy = '';

  from(table) { this.#table = table; return this; }
  where(condition) { this.#conditions.push(condition); return this; }
  order(field) { this.#orderBy = field; return this; }

  build() {
    let query = `SELECT * FROM ${this.#table}`;
    if (this.#conditions.length) query += ` WHERE ${this.#conditions.join(' AND ')}`;
    if (this.#orderBy) query += ` ORDER BY ${this.#orderBy}`;
    return query;
  }
}

const sql = new QueryBuilder()
  .from('users')
  .where('age > 18')
  .where('active = true')
  .order('name')
  .build();
// "SELECT * FROM users WHERE age > 18 AND active = true ORDER BY name"
```


## Resources

- [Refactoring Guru: Builder](https://refactoring.guru/design-patterns/builder) — Builder pattern guide

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*