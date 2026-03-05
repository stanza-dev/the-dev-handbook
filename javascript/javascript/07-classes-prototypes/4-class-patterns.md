---
source_course: "javascript"
source_lesson: "javascript-class-patterns"
---

# Class Patterns & Best Practices

## Introduction

Beyond basic class syntax, several patterns help write cleaner, more maintainable object-oriented JavaScript. From handling `this` in callbacks to implementing common design patterns, these techniques are essential for production code.

## Key Concepts

**Auto-binding**: Ensuring methods have correct `this` when used as callbacks.

**Factory Methods**: Static methods that create instances with custom logic.

**Immutable Classes**: Classes that don't allow modification after creation.

## Real World Context

React class components need bound methods. Data models often use factories. Service classes implement singleton patterns. These patterns appear throughout modern JavaScript codebases.

## Deep Dive

### Binding Methods for Callbacks

```javascript
class Counter {
  count = 0;
  
  // Problem: 'this' is lost in callbacks
  increment() {
    this.count++;
  }
  
  // Solution 1: Arrow function property (most common)
  decrement = () => {
    this.count--;
  };
}

const counter = new Counter();

// increment loses 'this'
setTimeout(counter.increment, 100);  // Error!

// decrement works (arrow function)
setTimeout(counter.decrement, 100);  // Works!

// Solution 2: Bind in constructor
class Counter2 {
  constructor() {
    this.increment = this.increment.bind(this);
  }
  increment() {
    this.count++;
  }
}
```

### Factory Pattern

```javascript
class User {
  constructor(data) {
    this.name = data.name;
    this.email = data.email;
    this.role = data.role;
  }
  
  // Factory methods
  static createAdmin(name, email) {
    return new User({ name, email, role: 'admin' });
  }
  
  static createGuest() {
    return new User({ name: 'Guest', email: '', role: 'guest' });
  }
  
  static async fromAPI(id) {
    const response = await fetch(`/api/users/${id}`);
    const data = await response.json();
    return new User(data);
  }
}

const admin = User.createAdmin('Alice', 'alice@admin.com');
const guest = User.createGuest();
const user = await User.fromAPI(123);
```

### Builder Pattern

```javascript
class QueryBuilder {
  #table;
  #conditions = [];
  #orderBy = null;
  #limit = null;
  
  from(table) {
    this.#table = table;
    return this;  // Enable chaining
  }
  
  where(condition) {
    this.#conditions.push(condition);
    return this;
  }
  
  order(field, direction = 'ASC') {
    this.#orderBy = `${field} ${direction}`;
    return this;
  }
  
  take(n) {
    this.#limit = n;
    return this;
  }
  
  build() {
    let sql = `SELECT * FROM ${this.#table}`;
    if (this.#conditions.length) {
      sql += ` WHERE ${this.#conditions.join(' AND ')}`;
    }
    if (this.#orderBy) sql += ` ORDER BY ${this.#orderBy}`;
    if (this.#limit) sql += ` LIMIT ${this.#limit}`;
    return sql;
  }
}

const query = new QueryBuilder()
  .from('users')
  .where('age > 18')
  .where('active = true')
  .order('name')
  .take(10)
  .build();
```

### Singleton Pattern

```javascript
class Database {
  static #instance = null;
  
  constructor() {
    if (Database.#instance) {
      return Database.#instance;
    }
    this.connection = this.connect();
    Database.#instance = this;
  }
  
  static getInstance() {
    if (!Database.#instance) {
      Database.#instance = new Database();
    }
    return Database.#instance;
  }
  
  connect() {
    console.log('Connecting to database...');
    return { connected: true };
  }
}

const db1 = Database.getInstance();
const db2 = Database.getInstance();
db1 === db2;  // true - same instance
```

### Immutable Class

```javascript
class Point {
  #x;
  #y;
  
  constructor(x, y) {
    this.#x = x;
    this.#y = y;
    Object.freeze(this);  // Prevent adding properties
  }
  
  get x() { return this.#x; }
  get y() { return this.#y; }
  
  // Return new instance instead of mutating
  move(dx, dy) {
    return new Point(this.#x + dx, this.#y + dy);
  }
  
  scale(factor) {
    return new Point(this.#x * factor, this.#y * factor);
  }
}

const p1 = new Point(1, 2);
const p2 = p1.move(3, 4);  // New point
p1.x;  // Still 1
p2.x;  // 4
```

## Common Pitfalls

1. **Forgetting to return `this` for chaining**: Builder pattern breaks.
2. **Singleton with side effects in constructor**: Can cause issues on import.
3. **Not freezing immutable objects**: They can still be modified.

## Best Practices

- **Use arrow properties for event handlers**: Auto-binds `this`.
- **Use factories for complex construction**: Cleaner than constructor overloading.
- **Return `this` for fluent APIs**: Enables method chaining.
- **Document class invariants**: Especially for immutable classes.

## Summary

Use arrow function properties to auto-bind methods. Factory methods provide flexible construction. Builder pattern enables fluent APIs. Singleton ensures single instances. Immutable classes return new instances instead of mutating. These patterns make classes more practical and maintainable.

## Code Examples

**Binding Methods for Callbacks**

```javascript
class Counter {
  count = 0;
  
  // Problem: 'this' is lost in callbacks
  increment() {
    this.count++;
  }
  
  // Solution 1: Arrow function property (most common)
  decrement = () => {
    this.count--;
  };
}

const counter = new Counter();

// increment loses 'this'
setTimeout(counter.increment, 100);  // Error!
```

**Factory Pattern**

```javascript
class User {
  constructor(data) {
    this.name = data.name;
    this.email = data.email;
    this.role = data.role;
  }
  
  // Factory methods
  static createAdmin(name, email) {
    return new User({ name, email, role: 'admin' });
  }
  
  static createGuest() {
    return new User({ name: 'Guest', email: '', role: 'guest' });
  }
  
  static async fromAPI(id) {
    const response = await fetch(`/api/users/${id}`);
```


## Resources

- [MDN: Using classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_classes) — Practical guide to using classes

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*