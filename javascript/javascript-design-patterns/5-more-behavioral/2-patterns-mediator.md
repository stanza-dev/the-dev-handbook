---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-mediator"
---

# The Mediator Pattern

## Introduction

The Mediator pattern defines an object that encapsulates how objects interact. It promotes loose coupling by keeping objects from referring to each other directly.

## Key Concepts

**Mediator**: The central hub that coordinates communication between colleagues.

**Colleague**: An object that communicates through the mediator instead of directly with other colleagues.

**Event Bus**: A common mediator implementation using publish/subscribe.

## Real World Context

Air traffic control is the classic analogy—planes don't communicate with each other, they talk to the tower. Chat rooms, Redux stores, and form validation coordinators are all mediators. MVC controllers often act as mediators between models and views.

## Deep Dive

### Chat Room

Users never reference each other directly. Instead, they communicate through the `ChatRoom` mediator, which routes messages to the correct recipients:

```javascript
class ChatRoom {
  #users = new Map();
  
  register(user) {
    this.#users.set(user.name, user);
    user.chatRoom = this;
  }
  
  send(message, from, to) {
    if (to) {
      // Private message
      this.#users.get(to)?.receive(message, from);
    } else {
      // Broadcast
      this.#users.forEach((user, name) => {
        if (name !== from) user.receive(message, from);
      });
    }
  }
}

class User {
  constructor(name) {
    this.name = name;
  }
  
  send(message, to) {
    this.chatRoom.send(message, this.name, to);
  }
  
  receive(message, from) {
    console.log(`${from} -> ${this.name}: ${message}`);
  }
}

const room = new ChatRoom();
const alice = new User('Alice');
const bob = new User('Bob');
room.register(alice);
room.register(bob);
alice.send('Hello!'); // Broadcasts
```

When Alice sends a message without specifying a recipient, the chat room broadcasts it to all other registered users. Private messages are routed to a single named user.

### Event Bus

An event bus is a lightweight mediator that uses publish/subscribe semantics. Components emit events without knowing who listens:

```javascript
const eventBus = {
  #handlers: new Map(),
  on(event, handler) {
    if (!this.#handlers.has(event)) this.#handlers.set(event, []);
    this.#handlers.get(event).push(handler);
  },
  emit(event, data) {
    this.#handlers.get(event)?.forEach(h => h(data));
  }
};
```

The event bus fully decouples publishers from subscribers. A component emitting `'userLoggedIn'` does not need to know which components are listening for that event.

## Common Pitfalls

1. **God mediator** — A mediator that knows too much about every colleague becomes a maintenance bottleneck.
2. **Hidden dependencies** — Communication through a mediator can obscure which components depend on which, making debugging harder.
3. **Single point of failure** — If the mediator goes down, all communication stops. Keep mediators simple and reliable.

## Best Practices

1. **Keep the mediator focused** — It should route messages, not contain business logic.
2. **Use typed events** — Define event names as constants to prevent typos and enable autocomplete.
3. **Provide unsubscribe mechanisms** — Like Observer, always allow colleagues to detach from the mediator.

## Summary

Mediator centralizes communication between objects. Reduces coupling—components don't know each other. Use for chat rooms, form validators, event buses.

## Code Examples

**Mediator as a chat room — users communicate through the room instead of referencing each other directly**

```javascript
class ChatRoom {
  #users = new Map();

  register(user) {
    this.#users.set(user.name, user);
    user.chatRoom = this;
  }

  send(message, from, to) {
    if (to) {
      this.#users.get(to)?.receive(message, from);
    } else {
      this.#users.forEach((user, name) => {
        if (name !== from) user.receive(message, from);
      });
    }
  }
}

class User {
  constructor(name) { this.name = name; }
  send(msg, to) { this.chatRoom.send(msg, this.name, to); }
  receive(msg, from) { console.log(`${from} → ${this.name}: ${msg}`); }
}
```


## Resources

- [Refactoring Guru: Mediator](https://refactoring.guru/design-patterns/mediator) — Mediator pattern guide

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*