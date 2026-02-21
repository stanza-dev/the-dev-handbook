---
source_course: "react-intermediate"
source_lesson: "react-reducer-with-context"
---

# Combining Reducer with Context

## Introduction

The `useReducer` + Context combination is one of React's most powerful built-in state management patterns. It gives you centralized state transitions (reducer), a predictable update mechanism (dispatch), and tree-wide state sharing (context) — all without external libraries. This pattern is ideal for medium-complexity applications before you need a dedicated state management library.

## Key Concepts

- **State + Dispatch Provider**: A context that provides both the current state and the dispatch function to the entire component tree.
- **Split Provider Pattern**: Separating state and dispatch into two contexts so components that only dispatch do not re-render when state changes.
- **Reducer + Context as Mini-Redux**: This pattern mirrors the core Redux architecture but with built-in React APIs.
- **Provider Encapsulation**: Wrapping the reducer, provider, and custom hooks in a single module for a clean API.

## Real World Context

A project management application has a board with columns (Todo, In Progress, Done) and draggable task cards. Many components need to read the board state (column headers, card counts, individual cards), and many components need to dispatch actions (drag-and-drop handlers, add/edit/delete buttons). The reducer + context pattern lets any component in the tree read or update the board state without prop drilling.

## Deep Dive

### The Full Pattern

```tsx
import { createContext, useContext, useReducer, type ReactNode } from 'react';

// 1. Define state and actions
type Task = { id: string; title: string; status: 'todo' | 'in-progress' | 'done' };
type BoardState = { tasks: Task[] };

type BoardAction =
  | { type: 'ADD_TASK'; payload: { title: string } }
  | { type: 'MOVE_TASK'; payload: { id: string; status: Task['status'] } }
  | { type: 'DELETE_TASK'; payload: { id: string } };

// 2. Write the reducer
function boardReducer(state: BoardState, action: BoardAction): BoardState {
  switch (action.type) {
    case 'ADD_TASK':
      return {
        tasks: [...state.tasks, {
          id: crypto.randomUUID(),
          title: action.payload.title,
          status: 'todo',
        }],
      };
    case 'MOVE_TASK':
      return {
        tasks: state.tasks.map(task =>
          task.id === action.payload.id
            ? { ...task, status: action.payload.status }
            : task
        ),
      };
    case 'DELETE_TASK':
      return { tasks: state.tasks.filter(t => t.id !== action.payload.id) };
    default:
      return state;
  }
}

// 3. Create contexts (split for performance)
const BoardStateContext = createContext<BoardState | null>(null);
const BoardDispatchContext = createContext<React.Dispatch<BoardAction> | null>(null);

// 4. Provider component
function BoardProvider({ children }: { children: ReactNode }) {
  const [state, dispatch] = useReducer(boardReducer, { tasks: [] });

  return (
    <BoardStateContext value={state}>
      <BoardDispatchContext value={dispatch}>
        {children}
      </BoardDispatchContext>
    </BoardStateContext>
  );
}

// 5. Custom hooks for consumers
function useBoardState() {
  const ctx = useContext(BoardStateContext);
  if (!ctx) throw new Error('useBoardState must be inside BoardProvider');
  return ctx;
}

function useBoardDispatch() {
  const ctx = useContext(BoardDispatchContext);
  if (!ctx) throw new Error('useBoardDispatch must be inside BoardProvider');
  return ctx;
}
```

### Using the Pattern

```tsx
function Column({ status }: { status: Task['status'] }) {
  const { tasks } = useBoardState(); // Only state — no dispatch
  const filtered = tasks.filter(t => t.status === status);

  return (
    <div className="column">
      <h2>{status}</h2>
      {filtered.map(task => <TaskCard key={task.id} task={task} />)}
    </div>
  );
}

function TaskCard({ task }: { task: Task }) {
  const dispatch = useBoardDispatch(); // Only dispatch — no state

  return (
    <div className="task-card">
      <span>{task.title}</span>
      <button onClick={() => dispatch({
        type: 'MOVE_TASK',
        payload: { id: task.id, status: 'done' },
      })}>
        Mark Done
      </button>
    </div>
  );
}

function App() {
  return (
    <BoardProvider>
      <div className="board">
        <Column status="todo" />
        <Column status="in-progress" />
        <Column status="done" />
      </div>
    </BoardProvider>
  );
}
```

### When to Use This vs External Libraries

| Use Case | Recommendation |
|----------|---------------|
| Small to medium app, few contexts | Reducer + Context |
| Large app, many cross-cutting concerns | Zustand, Jotai, or Redux Toolkit |
| Server state (API data) | TanStack Query or SWR |
| Simple prop drilling fix | Context alone (without reducer) |

## Common Pitfalls

1. **Not splitting state and dispatch contexts** — If all consumers get both state and dispatch from the same context, components that only dispatch (like a button) will re-render when state changes.
2. **Side effects inside the reducer** — Reducers must be pure. Dispatch an action, let the reducer update state, then use `useEffect` to trigger side effects based on the new state.

## Best Practices

1. **Export only the hooks, not the contexts** — Hide the raw contexts behind `useBoardState()` and `useBoardDispatch()` hooks. This enforces the null check and keeps the API clean.
2. **Co-locate reducer, contexts, and hooks in one file** — This creates a self-contained state module that is easy to understand and test independently.

## Summary

- Combining `useReducer` with Context provides centralized state management with tree-wide access.
- Split state and dispatch into separate contexts to minimize unnecessary re-renders.
- Encapsulate the reducer, contexts, and custom hooks in a single module for a clean, maintainable API.

## Code Examples

**Complete reducer + split context module with custom hooks**

```tsx
// Self-contained state module
const CounterStateCtx = createContext<{ count: number } | null>(null);
const CounterDispatchCtx = createContext<React.Dispatch<CounterAction> | null>(null);

type CounterAction = { type: 'inc' } | { type: 'dec' } | { type: 'reset' };

function counterReducer(state: { count: number }, action: CounterAction) {
  switch (action.type) {
    case 'inc': return { count: state.count + 1 };
    case 'dec': return { count: state.count - 1 };
    case 'reset': return { count: 0 };
  }
}

function CounterProvider({ children }: { children: React.ReactNode }) {
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });
  return (
    <CounterStateCtx value={state}>
      <CounterDispatchCtx value={dispatch}>{children}</CounterDispatchCtx>
    </CounterStateCtx>
  );
}

function useCounterState() {
  const ctx = useContext(CounterStateCtx);
  if (!ctx) throw new Error('Must be inside CounterProvider');
  return ctx;
}

function useCounterDispatch() {
  const ctx = useContext(CounterDispatchCtx);
  if (!ctx) throw new Error('Must be inside CounterProvider');
  return ctx;
}
```


## Resources

- [Scaling Up with Reducer and Context](https://react.dev/learn/scaling-up-with-reducer-and-context) — Official guide on combining useReducer with Context

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*