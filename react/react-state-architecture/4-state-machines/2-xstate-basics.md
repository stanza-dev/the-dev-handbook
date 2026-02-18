---
source_course: "react-state-architecture"
source_lesson: "react-state-architecture-xstate-basics"
---

# XState for Complex Logic

XState is a powerful state machine library.

## Installation

```bash
npm install xstate @xstate/react
```

## Basic Machine

```jsx
import { createMachine, assign } from 'xstate';
import { useMachine } from '@xstate/react';

const toggleMachine = createMachine({
  id: 'toggle',
  initial: 'inactive',
  states: {
    inactive: {
      on: { TOGGLE: 'active' },
    },
    active: {
      on: { TOGGLE: 'inactive' },
    },
  },
});

function Toggle() {
  const [state, send] = useMachine(toggleMachine);
  
  return (
    <button onClick={() => send({ type: 'TOGGLE' })}>
      {state.matches('active') ? 'ON' : 'OFF'}
    </button>
  );
}
```

## Machine with Context

```jsx
const counterMachine = createMachine({
  id: 'counter',
  initial: 'active',
  context: {
    count: 0,
  },
  states: {
    active: {
      on: {
        INCREMENT: {
          actions: assign({
            count: ({ context }) => context.count + 1,
          }),
        },
        DECREMENT: {
          actions: assign({
            count: ({ context }) => context.count - 1,
          }),
        },
      },
    },
  },
});

function Counter() {
  const [state, send] = useMachine(counterMachine);
  
  return (
    <div>
      <span>{state.context.count}</span>
      <button onClick={() => send({ type: 'INCREMENT' })}>+</button>
      <button onClick={() => send({ type: 'DECREMENT' })}>-</button>
    </div>
  );
}
```

## Async Actions

```jsx
const fetchMachine = createMachine({
  id: 'fetch',
  initial: 'idle',
  context: {
    data: null,
    error: null,
  },
  states: {
    idle: {
      on: { FETCH: 'loading' },
    },
    loading: {
      invoke: {
        src: 'fetchData',
        onDone: {
          target: 'success',
          actions: assign({
            data: ({ event }) => event.output,
          }),
        },
        onError: {
          target: 'failure',
          actions: assign({
            error: ({ event }) => event.error,
          }),
        },
      },
    },
    success: {
      on: { FETCH: 'loading' },
    },
    failure: {
      on: { RETRY: 'loading' },
    },
  },
});

function DataFetcher({ url }) {
  const [state, send] = useMachine(fetchMachine, {
    actors: {
      fetchData: async () => {
        const response = await fetch(url);
        return response.json();
      },
    },
  });
  
  if (state.matches('idle')) {
    return <button onClick={() => send({ type: 'FETCH' })}>Load</button>;
  }
  if (state.matches('loading')) {
    return <div>Loading...</div>;
  }
  if (state.matches('failure')) {
    return (
      <div>
        Error: {state.context.error}
        <button onClick={() => send({ type: 'RETRY' })}>Retry</button>
      </div>
    );
  }
  return <div>{JSON.stringify(state.context.data)}</div>;
}
```

## Guards (Conditional Transitions)

```jsx
const paymentMachine = createMachine({
  id: 'payment',
  initial: 'idle',
  context: {
    amount: 0,
    balance: 100,
  },
  states: {
    idle: {
      on: {
        PAY: {
          target: 'processing',
          guard: ({ context }) => context.amount <= context.balance,
        },
      },
    },
    processing: {
      // ...
    },
  },
});
```

## Visualizing Machines

Use [Stately](https://stately.ai/viz) to visualize your state machines:

```jsx
// https://stately.ai/viz
// Paste your machine definition to see the diagram
```

---

> 📘 *This lesson is part of the [React State Architecture](https://stanza.dev/courses/react-state-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*