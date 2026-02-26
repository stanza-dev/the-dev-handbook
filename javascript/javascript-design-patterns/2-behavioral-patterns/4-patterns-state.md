---
source_course: "javascript-design-patterns"
source_lesson: "javascript-design-patterns-state"
---

# The State Pattern

## Introduction

The State pattern allows an object to alter its behavior when its internal state changes. Perfect for state machines.

## Key Concepts

**State Object**: Encapsulates behavior associated with a particular state.

**Context**: The object whose behavior changes as its state transitions.

**Transition**: Moving from one state to another in response to an event.

**State Machine**: A formal model of all states and valid transitions.

## Real World Context

UI workflows (checkout flows, multi-step forms), network connections (connecting → connected → disconnected), game character behavior, and media players all use the State pattern. Libraries like XState formalize this into finite state machines.

## Deep Dive

### Traffic Light

Each state object defines its own `next()` transition, so the traffic light simply delegates state changes without any conditional logic:

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

The `TrafficLight` class has no `if/else` or `switch` statements. All transition logic lives in the state objects themselves, making it trivial to add or modify states.

### Form State Machine

A form state machine restricts transitions to only valid actions for each state. Invalid actions are silently rejected by returning `false`:

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

Notice that the `submitted` state has no transitions at all, making it a terminal state. The `error` state only allows `retry`, preventing other actions while in the error state.

## Common Pitfalls

1. **Missing transitions** — Forgetting to define an edge case transition can leave the system stuck in an invalid state.
2. **Too many states** — State explosion makes the machine hard to reason about; group related states into hierarchical machines.
3. **Side effects in transitions** — Keep state transitions pure; trigger side effects in the context after the transition completes.

## Best Practices

1. **Enumerate all valid states upfront** — Define an explicit list of states so invalid states are impossible.
2. **Make invalid transitions throw** — Attempting an undefined transition should fail loudly, not silently.
3. **Visualize the state machine** — A diagram of states and transitions is the best documentation for this pattern.

## Summary

State pattern encapsulates state-dependent behavior. Each state knows valid transitions. Great for workflows and UI states.

## Code Examples

**State pattern as a traffic light — each state object defines its own transition via next()**

```javascript
const states = {
  green:  { color: 'green',  next: () => states.yellow },
  yellow: { color: 'yellow', next: () => states.red },
  red:    { color: 'red',    next: () => states.green }
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
light.change(); // 'green'
```


## Resources

- [Refactoring Guru: State](https://refactoring.guru/design-patterns/state) — State pattern guide

---

> 📘 *This lesson is part of the [JavaScript Design Patterns](https://stanza.dev/courses/javascript-design-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*