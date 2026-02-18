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

### Factory with Configuration

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

### Factory Functions (Functional Style)

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

### Abstract Factory

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

## Resources

- [Patterns.dev: Factory Pattern](https://www.patterns.dev/posts/factory-pattern/) — Factory pattern guide
- [Refactoring Guru: Factory Method](https://refactoring.guru/design-patterns/factory-method) — Factory Method pattern

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*