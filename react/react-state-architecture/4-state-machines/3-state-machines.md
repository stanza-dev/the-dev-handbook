---
source_course: "react-state-architecture"
source_lesson: "react-state-architecture-ui-state-machines"
---

# UI State Machine Examples

Practical state machines for common UI patterns.

## Form Submission Machine

```jsx
const formMachine = createMachine({
  id: 'form',
  initial: 'editing',
  context: {
    values: {},
    errors: {},
  },
  states: {
    editing: {
      on: {
        CHANGE: {
          actions: assign({
            values: ({ context, event }) => ({
              ...context.values,
              [event.field]: event.value,
            }),
          }),
        },
        SUBMIT: {
          target: 'validating',
        },
      },
    },
    validating: {
      invoke: {
        src: 'validate',
        onDone: [
          {
            target: 'submitting',
            guard: ({ event }) => Object.keys(event.output).length === 0,
          },
          {
            target: 'editing',
            actions: assign({
              errors: ({ event }) => event.output,
            }),
          },
        ],
      },
    },
    submitting: {
      invoke: {
        src: 'submitForm',
        onDone: 'success',
        onError: {
          target: 'editing',
          actions: assign({
            errors: ({ event }) => ({ submit: event.error.message }),
          }),
        },
      },
    },
    success: {
      type: 'final',
    },
  },
});
```

## Multi-Step Wizard

```jsx
const wizardMachine = createMachine({
  id: 'wizard',
  initial: 'step1',
  context: {
    step1Data: null,
    step2Data: null,
    step3Data: null,
  },
  states: {
    step1: {
      on: {
        NEXT: {
          target: 'step2',
          actions: assign({
            step1Data: ({ event }) => event.data,
          }),
        },
      },
    },
    step2: {
      on: {
        BACK: 'step1',
        NEXT: {
          target: 'step3',
          actions: assign({
            step2Data: ({ event }) => event.data,
          }),
        },
      },
    },
    step3: {
      on: {
        BACK: 'step2',
        SUBMIT: 'submitting',
      },
    },
    submitting: {
      invoke: {
        src: 'submitWizard',
        onDone: 'complete',
        onError: 'step3',
      },
    },
    complete: {
      type: 'final',
    },
  },
});

function Wizard() {
  const [state, send] = useMachine(wizardMachine);
  
  return (
    <div>
      {state.matches('step1') && (
        <Step1
          onNext={(data) => send({ type: 'NEXT', data })}
        />
      )}
      {state.matches('step2') && (
        <Step2
          onBack={() => send({ type: 'BACK' })}
          onNext={(data) => send({ type: 'NEXT', data })}
        />
      )}
      {state.matches('step3') && (
        <Step3
          onBack={() => send({ type: 'BACK' })}
          onSubmit={() => send({ type: 'SUBMIT' })}
        />
      )}
      {state.matches('submitting') && <Loading />}
      {state.matches('complete') && <Success />}
    </div>
  );
}
```

## Modal with Animation States

```jsx
const modalMachine = createMachine({
  id: 'modal',
  initial: 'closed',
  states: {
    closed: {
      on: { OPEN: 'opening' },
    },
    opening: {
      after: {
        300: 'open', // Transition after animation
      },
    },
    open: {
      on: { CLOSE: 'closing' },
    },
    closing: {
      after: {
        200: 'closed',
      },
    },
  },
});

function Modal({ children }) {
  const [state, send] = useMachine(modalMachine);
  
  if (state.matches('closed')) return null;
  
  return (
    <div
      className={`modal ${state.value}`}
      onClick={() => send({ type: 'CLOSE' })}
    >
      {children}
    </div>
  );
}
```

## When to Use State Machines

✅ **Good use cases:**
- Forms with validation
- Multi-step wizards
- Async workflows
- Complex modals
- Drag and drop
- Authentication flows

❌ **Overkill for:**
- Simple toggles
- Basic counters
- Single-state components

---

> 📘 *This lesson is part of the [React State Architecture](https://stanza.dev/courses/react-state-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*