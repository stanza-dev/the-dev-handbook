---
source_course: "typescript-architecture"
source_lesson: "typescript-architecture-observer-pattern"
---

# Typed Observer & EventEmitter

## Introduction
The Observer pattern lets objects subscribe to events and get notified when something happens, without tight coupling between the publisher and subscribers. TypeScript generics can make event emitters fully type-safe: the compiler knows exactly which events exist and what data each event carries.

## Key Concepts
- **Observer Pattern**: A publisher maintains a list of subscribers and notifies them when an event occurs.
- **EventMap**: A type that maps event names to their payload types, providing compile-time safety for event registration and emission.
- **Generic EventEmitter**: An emitter typed with an EventMap so that `on("click", handler)` only accepts handlers matching the click event's payload.

## Real World Context
Node.js `EventEmitter`, DOM event listeners, React state libraries, and message buses all use the Observer pattern. A typed EventMap prevents subscribing to nonexistent events or passing the wrong handler signature.

## Deep Dive
### Defining an EventMap

```typescript
interface EventMap {
    userLoggedIn: { userId: string; timestamp: number };
    orderPlaced: { orderId: string; total: number };
    error: { message: string; code: number };
}
```

### Generic Typed Emitter

```typescript
class TypedEmitter<Events extends Record<string, unknown>> {
    private listeners: {
        [K in keyof Events]?: Array<(payload: Events[K]) => void>;
    } = {};

    on<K extends keyof Events>(event: K, handler: (payload: Events[K]) => void): void {
        (this.listeners[event] ??= []).push(handler);
    }

    off<K extends keyof Events>(event: K, handler: (payload: Events[K]) => void): void {
        const list = this.listeners[event];
        if (list) {
            this.listeners[event] = list.filter((h) => h !== handler) as any;
        }
    }

    emit<K extends keyof Events>(event: K, payload: Events[K]): void {
        this.listeners[event]?.forEach((handler) => handler(payload));
    }
}
```

### Usage

```typescript
const bus = new TypedEmitter<EventMap>();

bus.on("userLoggedIn", ({ userId, timestamp }) => {
    // TypeScript knows userId is string, timestamp is number
    console.log(`User ${userId} logged in at ${timestamp}`);
});

bus.emit("userLoggedIn", { userId: "u-42", timestamp: Date.now() }); // OK
// bus.emit("userLoggedIn", { userId: 42 }); // Error: number is not string
// bus.on("unknown", () => {}); // Error: 'unknown' is not in EventMap
```

## Common Pitfalls
1. **Forgetting to remove listeners** — Memory leaks happen when listeners are registered but never removed. Always call `off()` in cleanup logic.
2. **Loose event names** — Using `string` for event names loses type safety. Always define an EventMap interface.

## Best Practices
1. **Define EventMaps as interfaces** — This allows module augmentation to extend event types across packages.
2. **Return unsubscribe functions** — Instead of requiring callers to keep a reference to the handler, return a cleanup function from `on()`.

## Summary
- The Observer pattern decouples event publishers from subscribers.
- A typed EventMap ensures compile-time safety for event names and payloads.
- Generic EventEmitter enforces that handlers match the payload type of the event.
- Always provide cleanup mechanisms to prevent memory leaks.

## Code Examples

**A fully typed EventEmitter that returns an unsubscribe function — event names and payloads are enforced at compile time**

```typescript
interface AppEvents {
    navigate: { path: string };
    notify: { message: string; level: "info" | "error" };
}

class TypedEmitter<E extends Record<string, unknown>> {
    private listeners: Partial<{ [K in keyof E]: ((p: E[K]) => void)[] }> = {};

    on<K extends keyof E>(event: K, fn: (payload: E[K]) => void): () => void {
        (this.listeners[event] ??= [] as any).push(fn);
        return () => this.off(event, fn); // Return unsubscribe function
    }

    off<K extends keyof E>(event: K, fn: (payload: E[K]) => void): void {
        this.listeners[event] = this.listeners[event]?.filter(h => h !== fn) as any;
    }

    emit<K extends keyof E>(event: K, payload: E[K]): void {
        this.listeners[event]?.forEach(fn => fn(payload));
    }
}

const emitter = new TypedEmitter<AppEvents>();
const unsub = emitter.on("notify", ({ message, level }) => {
    console.log(`[${level}] ${message}`);
});
unsub(); // Clean up
```


## Resources

- [Node.js Events Documentation](https://nodejs.org/api/events.html) — Official Node.js documentation on the EventEmitter class

---

> 📘 *This lesson is part of the [TypeScript Architecture & Patterns](https://stanza.dev/courses/typescript-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*