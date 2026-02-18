---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-state"
---

# The State Pattern

## Introduction

The State pattern allows an object to alter its behavior when its internal state changes. Perfect for state machines.

## Deep Dive

### Traffic Light

```javascript
const states = {
  green: { color: 'green', next: () => states.yellow },
  yellow: { color: 'yellow', next: () => states.red },
  red: { color: 'red', next: () => states.green }
};

class TrafficLight {
  #state = states.green;
  
  get color() { return this.#state.color; }
  
  change() {
    this.#state = this.#state.next();
    return this.#state.color;
  }
}

const light = new TrafficLight();
light.change(); // 'yellow'
light.change(); // 'red'
```

### Form State Machine

```javascript
const formStates = {
  idle: { submit: () => formStates.submitting },
  submitting: {
    success: () => formStates.submitted,
    error: () => formStates.error
  },
  submitted: {},
  error: { retry: () => formStates.submitting }
};

class FormStateMachine {
  #state = formStates.idle;
  
  transition(action) {
    const next = this.#state[action];
    if (next) { this.#state = next(); return true; }
    return false;
  }
}
```

## Summary

State pattern encapsulates state-dependent behavior. Each state knows valid transitions. Great for workflows and UI states.

## Resources

- [Refactoring Guru: State](https://refactoring.guru/design-patterns/state) — State pattern guide

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*