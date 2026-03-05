---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-bridge"
---

# The Bridge Pattern

## Introduction

The Bridge pattern separates an abstraction from its implementation so they can vary independently. It's useful when both can have multiple variations.

## Key Concepts

**Abstraction**: The high-level control layer (e.g., RemoteControl).

**Implementation**: The low-level platform-specific code (e.g., TV, Radio).

**Decoupling**: Abstraction and implementation vary independently through composition, not inheritance.

## Real World Context

Cross-platform UI frameworks use Bridge to separate widget logic from platform rendering (e.g., React Native's bridge to iOS/Android). Database drivers use Bridge—your ORM logic is the abstraction, the PostgreSQL/MySQL driver is the implementation.

## Deep Dive

### Device Remote Control

The abstraction (`RemoteControl`) and implementation (`TV`, `Radio`) vary independently. Any remote can control any device through the shared interface:

```javascript
// Implementation
class TV {
  on() { console.log('TV on'); }
  off() { console.log('TV off'); }
  setVolume(v) { console.log(`TV volume: ${v}`); }
}

class Radio {
  on() { console.log('Radio on'); }
  off() { console.log('Radio off'); }
  setVolume(v) { console.log(`Radio volume: ${v}`); }
}

// Abstraction
class RemoteControl {
  constructor(device) {
    this.device = device;
  }
  
  togglePower() {
    this.isOn ? this.device.off() : this.device.on();
    this.isOn = !this.isOn;
  }
}

class AdvancedRemote extends RemoteControl {
  mute() {
    this.device.setVolume(0);
  }
}

// Use any remote with any device
const tvRemote = new AdvancedRemote(new TV());
const radioRemote = new RemoteControl(new Radio());
```

Adding a new device (e.g., `Speaker`) requires only a new implementation class. Adding a new remote type (e.g., `VoiceRemote`) requires only a new abstraction subclass. Neither side affects the other.

## Common Pitfalls

1. **Confusing with Adapter** — Adapter makes existing incompatible interfaces work together; Bridge is designed upfront to separate abstraction from implementation.
2. **Over-abstraction** — If you only have one implementation, Bridge adds unnecessary complexity.
3. **Leaking implementation details** — The abstraction should not expose implementation-specific methods.

## Best Practices

1. **Identify independent dimensions of variation** — Bridge is valuable when both the abstraction and implementation have multiple variants.
2. **Inject the implementation** — Pass the implementation to the abstraction's constructor (dependency injection).
3. **Define a clear implementation interface** — All implementations must satisfy the same contract.

## Summary

Bridge separates abstraction from implementation. Both can vary independently. Use when you have multiple dimensions of variation.

## Code Examples

**Bridge pattern — RemoteControl (abstraction) and TV/Radio (implementations) vary independently**

```javascript
// Implementation interface
class TV {
  on()  { console.log('TV on'); }
  off() { console.log('TV off'); }
  setVolume(v) { console.log(`TV volume: ${v}`); }
}

class Radio {
  on()  { console.log('Radio on'); }
  off() { console.log('Radio off'); }
  setVolume(v) { console.log(`Radio volume: ${v}`); }
}

// Abstraction — takes any device implementation
class RemoteControl {
  constructor(device) { this.device = device; }
  togglePower() { this.isOn ? this.device.off() : this.device.on(); this.isOn = !this.isOn; }
}

class AdvancedRemote extends RemoteControl {
  mute() { this.device.setVolume(0); }
}

new AdvancedRemote(new TV()).mute(); // "TV volume: 0"
```


## Resources

- [Refactoring Guru: Bridge](https://refactoring.guru/design-patterns/bridge) — Bridge pattern guide

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*