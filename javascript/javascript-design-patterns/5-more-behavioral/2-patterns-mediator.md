---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-mediator"
---

# The Mediator Pattern

## Introduction

The Mediator pattern defines an object that encapsulates how objects interact. It promotes loose coupling by keeping objects from referring to each other directly.

## Deep Dive

### Chat Room

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

### Event Bus

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

## Summary

Mediator centralizes communication between objects. Reduces coupling—components don't know each other. Use for chat rooms, form validators, event buses.

## Resources

- [Refactoring Guru: Mediator](https://refactoring.guru/design-patterns/mediator) — Mediator pattern guide

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*