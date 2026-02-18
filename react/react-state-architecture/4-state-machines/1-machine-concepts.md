---
source_course: "react-state-architecture"
source_lesson: "react-state-architecture-state-machine-concepts"
---

# State Machine Concepts

State machines model explicit states and transitions.

## The Problem: Boolean Soup

```jsx
// ❌ Hard to reason about
function DataLoader() {
  const [isLoading, setIsLoading] = useState(false);
  const [isError, setIsError] = useState(false);
  const [isSuccess, setIsSuccess] = useState(false);
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  
  // What if isLoading AND isError are both true?
  // What states are actually valid?
}
```

## State Machine Solution

```jsx
// ✅ Explicit states
const states = {
  IDLE: 'idle',
  LOADING: 'loading',
  SUCCESS: 'success',
  ERROR: 'error',
};

function DataLoader() {
  const [state, setState] = useState(states.IDLE);
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  
  // Clear which state we're in
  // Impossible to be loading AND error
}
```

## Core Concepts

### States
Finite set of possible conditions:
```
idle → loading → success
                ↘ error
```

### Events/Actions
Triggers that cause transitions:
- FETCH
- RESOLVE
- REJECT
- RETRY

### Transitions
Rules for moving between states:
```
idle + FETCH → loading
loading + RESOLVE → success
loading + REJECT → error
error + RETRY → loading
```

## Simple State Machine Pattern

```jsx
const STATES = {
  idle: 'idle',
  loading: 'loading',
  success: 'success',
  error: 'error',
};

const EVENTS = {
  FETCH: 'FETCH',
  RESOLVE: 'RESOLVE',
  REJECT: 'REJECT',
  RETRY: 'RETRY',
};

function reducer(state, event) {
  switch (state.status) {
    case STATES.idle:
      if (event.type === EVENTS.FETCH) {
        return { status: STATES.loading };
      }
      return state;
      
    case STATES.loading:
      if (event.type === EVENTS.RESOLVE) {
        return { status: STATES.success, data: event.data };
      }
      if (event.type === EVENTS.REJECT) {
        return { status: STATES.error, error: event.error };
      }
      return state;
      
    case STATES.success:
      if (event.type === EVENTS.FETCH) {
        return { status: STATES.loading };
      }
      return state;
      
    case STATES.error:
      if (event.type === EVENTS.RETRY) {
        return { status: STATES.loading };
      }
      return state;
      
    default:
      return state;
  }
}

function useDataFetcher(fetchFn) {
  const [state, dispatch] = useReducer(reducer, { status: STATES.idle });
  
  const fetch = async () => {
    dispatch({ type: EVENTS.FETCH });
    try {
      const data = await fetchFn();
      dispatch({ type: EVENTS.RESOLVE, data });
    } catch (error) {
      dispatch({ type: EVENTS.REJECT, error });
    }
  };
  
  const retry = () => {
    fetch();
  };
  
  return { state, fetch, retry };
}
```

## Benefits of State Machines

1. **Impossible states are impossible**
2. **Self-documenting logic**
3. **Easier to test**
4. **Predictable behavior**
5. **Visual debugging**

---

> 📘 *This lesson is part of the [React State Architecture](https://stanza.dev/courses/react-state-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*