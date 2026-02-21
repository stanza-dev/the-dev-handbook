---
source_course: "react-intermediate"
source_lesson: "react-usereducer-basics"
---

# useReducer Basics

## Introduction

While `useState` works well for simple state (a counter, a form field, a boolean toggle), it becomes unwieldy when state has multiple sub-values with complex update logic. The `useReducer` hook provides an alternative that centralizes state transitions in a pure reducer function, making complex state predictable and testable.

## Key Concepts

- **Reducer Function**: A pure function `(state, action) => newState` that takes the current state and an action, and returns the new state.
- **Dispatch Function**: The function returned by `useReducer` that sends actions to the reducer.
- **Action**: An object (typically with a `type` property) that describes what happened. May include a `payload` with additional data.
- **Initial State**: The starting state value passed as the second argument to `useReducer`.

## Real World Context

A multi-step checkout form has state for the current step, form data for shipping/billing/payment, validation errors, and a loading flag for the submission. Using `useState` for each piece means coordinating updates across 5+ setter functions. A reducer centralizes all these transitions: `NEXT_STEP`, `PREV_STEP`, `UPDATE_FIELD`, `SET_ERROR`, `SUBMIT_START`, `SUBMIT_SUCCESS`. Each action is a clear, declarative description of what happened.

## Deep Dive

### Basic Syntax

```tsx
const [state, dispatch] = useReducer(reducer, initialState);
```

### Example: Task Manager

```tsx
import { useReducer } from 'react';

type Task = { id: number; text: string; done: boolean };

type State = { tasks: Task[]; filter: 'all' | 'active' | 'completed' };

type Action =
  | { type: 'ADD_TASK'; payload: string }
  | { type: 'TOGGLE_TASK'; payload: number }
  | { type: 'DELETE_TASK'; payload: number }
  | { type: 'SET_FILTER'; payload: State['filter'] };

function taskReducer(state: State, action: Action): State {
  switch (action.type) {
    case 'ADD_TASK':
      return {
        ...state,
        tasks: [...state.tasks, {
          id: Date.now(),
          text: action.payload,
          done: false,
        }],
      };
    case 'TOGGLE_TASK':
      return {
        ...state,
        tasks: state.tasks.map(task =>
          task.id === action.payload
            ? { ...task, done: !task.done }
            : task
        ),
      };
    case 'DELETE_TASK':
      return {
        ...state,
        tasks: state.tasks.filter(task => task.id !== action.payload),
      };
    case 'SET_FILTER':
      return { ...state, filter: action.payload };
    default:
      return state;
  }
}

function TaskManager() {
  const [state, dispatch] = useReducer(taskReducer, {
    tasks: [],
    filter: 'all',
  });

  const [newTask, setNewTask] = useState('');

  const handleAddTask = (e: React.FormEvent) => {
    e.preventDefault();
    if (!newTask.trim()) return;
    dispatch({ type: 'ADD_TASK', payload: newTask.trim() });
    setNewTask('');
  };

  const visibleTasks = state.tasks.filter(task => {
    if (state.filter === 'active') return !task.done;
    if (state.filter === 'completed') return task.done;
    return true;
  });

  return (
    <div>
      <form onSubmit={handleAddTask}>
        <input value={newTask} onChange={e => setNewTask(e.target.value)} />
        <button type="submit">Add</button>
      </form>
      <div>
        {(['all', 'active', 'completed'] as const).map(f => (
          <button key={f} onClick={() => dispatch({ type: 'SET_FILTER', payload: f })}>
            {f}
          </button>
        ))}
      </div>
      <ul>
        {visibleTasks.map(task => (
          <li key={task.id}>
            <span
              onClick={() => dispatch({ type: 'TOGGLE_TASK', payload: task.id })}
              style={{ textDecoration: task.done ? 'line-through' : 'none' }}
            >
              {task.text}
            </span>
            <button onClick={() => dispatch({ type: 'DELETE_TASK', payload: task.id })}>
              Delete
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

### Why Reducers Are Testable

Since the reducer is a pure function with no side effects, you can test it without rendering components:

```tsx
test('ADD_TASK adds a new task', () => {
  const state = { tasks: [], filter: 'all' as const };
  const newState = taskReducer(state, { type: 'ADD_TASK', payload: 'Test' });
  expect(newState.tasks).toHaveLength(1);
  expect(newState.tasks[0].text).toBe('Test');
  expect(newState.tasks[0].done).toBe(false);
});
```

## Common Pitfalls

1. **Using useReducer for simple state** — A single boolean toggle or a text input does not benefit from a reducer. The boilerplate of action types and a switch statement adds complexity without value. Use `useState` for simple cases.
2. **Mutating state inside the reducer** — Reducers must return new objects, not mutate the existing state. `state.tasks.push(task)` is wrong; `[...state.tasks, task]` is correct.

## Best Practices

1. **Use TypeScript discriminated unions for actions** — Define a union type for all possible actions. This gives you exhaustive type checking in the switch statement and catches typos in action types.
2. **Keep reducers pure** — No API calls, no random values, no side effects inside a reducer. If you need side effects, trigger them in the component after dispatch.

## Summary

- `useReducer` centralizes complex state transitions in a pure reducer function, making them predictable and testable.
- Dispatch actions (objects with a `type` and optional `payload`) to trigger state transitions.
- Use reducers when state has multiple sub-values or when updates follow complex business rules.

## Code Examples

**A typed reducer with multiple action types and payloads**

```tsx
import { useReducer } from 'react';

type State = { count: number };
type Action =
  | { type: 'increment' }
  | { type: 'decrement' }
  | { type: 'incrementBy'; payload: number }
  | { type: 'reset' };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'increment': return { count: state.count + 1 };
    case 'decrement': return { count: state.count - 1 };
    case 'incrementBy': return { count: state.count + action.payload };
    case 'reset': return { count: 0 };
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });
  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+1</button>
      <button onClick={() => dispatch({ type: 'incrementBy', payload: 5 })}>+5</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
    </div>
  );
}
```


## Resources

- [useReducer Documentation](https://react.dev/reference/react/useReducer) — Official useReducer API reference
- [Extracting State Logic into a Reducer](https://react.dev/learn/extracting-state-logic-into-a-reducer) — Official guide on when and how to use reducers

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*