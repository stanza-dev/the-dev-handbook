---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-factory"
---

# The Factory Pattern

## Introduction

The Factory pattern creates objects without exposing the creation logic. Instead of calling constructors directly, you call a factory method that decides what to create based on parameters. This provides flexibility and decouples code from specific implementations.

## Key Concepts

**Factory Method**: A method that creates and returns objects.

**Product**: The object created by the factory.

**Decoupling**: Client code doesn't know about concrete classes.

## Real World Context

UI component libraries (create buttons/inputs), database drivers (create connections for MySQL/Postgres), notification systems (create email/SMS/push)—factories abstract creation complexity.

## Deep Dive

### Simple Factory

A simple factory uses a static method with a switch statement to create the right object based on a type parameter:

```javascript
class Car {
  constructor(type) {
    this.type = type;
  }
  drive() { return `Driving ${this.type}`; }
}

class Truck {
  constructor(type) {
    this.type = type;
  }
  drive() { return `Hauling with ${this.type}`; }
}

class VehicleFactory {
  static create(type) {
    switch (type) {
      case 'car': return new Car('sedan');
      case 'truck': return new Truck('pickup');
      default: throw new Error(`Unknown vehicle: ${type}`);
    }
  }
}

const vehicle = VehicleFactory.create('car');
vehicle.drive();  // 'Driving sedan'
```

The caller never uses `new Car()` or `new Truck()` directly. It only knows about `VehicleFactory.create()`, keeping the concrete classes hidden.

### Factory with Configuration

Factories can accept configuration objects to pass settings to each product, making creation more flexible:

```javascript
class NotificationFactory {
  static create(channel, options = {}) {
    const factories = {
      email: () => new EmailNotification(options.smtp),
      sms: () => new SMSNotification(options.provider),
      push: () => new PushNotification(options.service),
    };
    
    const factory = factories[channel];
    if (!factory) {
      throw new Error(`Unknown channel: ${channel}`);
    }
    return factory();
  }
}

const notifier = NotificationFactory.create('email', {
  smtp: { host: 'smtp.example.com' }
});
```

Using an object map instead of a switch statement keeps the factory open for extension. New channels can be added by simply inserting a new key.

### Factory Functions (Functional Style)

Factory functions return plain objects without `new` or classes, making them the most lightweight factory approach in JavaScript:

```javascript
const createUser = (name, role = 'user') => ({
  name,
  role,
  permissions: role === 'admin' 
    ? ['read', 'write', 'delete'] 
    : ['read'],
  greet() {
    return `Hello, I'm ${this.name}`;
  }
});

const user = createUser('Alice');
const admin = createUser('Bob', 'admin');
```

Note how the `role` parameter controls what permissions the returned object receives. No classes are involved, just closures and object literals.

### Abstract Factory

An Abstract Factory creates families of related objects that belong together. Here, each theme produces a matching button and input:

```javascript
// Family of related objects
const DarkTheme = {
  createButton: () => ({ style: 'dark', render() { /*...*/ } }),
  createInput: () => ({ style: 'dark', render() { /*...*/ } }),
};

const LightTheme = {
  createButton: () => ({ style: 'light', render() { /*...*/ } }),
  createInput: () => ({ style: 'light', render() { /*...*/ } }),
};

function createUI(theme) {
  return {
    button: theme.createButton(),
    input: theme.createInput()
  };
}

const ui = createUI(DarkTheme);
```

Swapping `DarkTheme` for `LightTheme` changes all created components at once. The abstract factory guarantees that button and input styles are always consistent.

## Common Pitfalls

1. **Overuse**: Not everything needs a factory.
2. **Giant switch statements**: Consider using object maps instead.
3. **Tight coupling to factory**: Can still create dependency issues.

## Best Practices

- **Use when creation is complex**: Multiple steps, configuration.
- **Use for polymorphism**: When type is determined at runtime.
- **Keep factories focused**: Single responsibility.
- **Consider factory functions**: Often simpler than classes.

## Summary

Factories create objects without exposing creation logic. Use when object creation is complex or type depends on runtime conditions. Factory functions are often simpler than class-based factories. Abstract factories create families of related objects.

## Code Examples

**Factory function that creates notification objects — the caller doesn't need to know how each type is built**

```javascript
function createNotification(type, message) {
  const base = { message, timestamp: Date.now() };
  switch (type) {
    case 'email':
      return { ...base, send: () => sendEmail(base.message) };
    case 'sms':
      return { ...base, send: () => sendSMS(base.message) };
    case 'push':
      return { ...base, send: () => sendPush(base.message) };
    default:
      throw new Error(`Unknown type: ${type}`);
  }
}

const notification = createNotification('email', 'Hello!');
notification.send();
```


## Resources

- [Patterns.dev: Factory Pattern](https://www.patterns.dev/posts/factory-pattern/) — Factory pattern guide
- [Refactoring Guru: Factory Method](https://refactoring.guru/design-patterns/factory-method) — Factory Method pattern

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*