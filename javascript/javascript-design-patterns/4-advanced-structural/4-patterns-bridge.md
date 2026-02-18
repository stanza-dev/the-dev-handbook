---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-bridge"
---

# The Bridge Pattern

## Introduction

The Bridge pattern separates an abstraction from its implementation so they can vary independently. It's useful when both can have multiple variations.

## Deep Dive

### Device Remote Control

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

## Summary

Bridge separates abstraction from implementation. Both can vary independently. Use when you have multiple dimensions of variation.

## Resources

- [Refactoring Guru: Bridge](https://refactoring.guru/design-patterns/bridge) — Bridge pattern guide

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*