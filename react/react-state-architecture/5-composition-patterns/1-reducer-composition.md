---
source_course: "react-state-architecture"
source_lesson: "react-state-architecture-reducer-composition"
---

# Reducer Composition

Combine reducers for modular state management.

## Combining Reducers

```jsx
function usersReducer(state, action) {
  switch (action.type) {
    case 'SET_USERS':
      return action.payload;
    case 'ADD_USER':
      return [...state, action.payload];
    default:
      return state;
  }
}

function postsReducer(state, action) {
  switch (action.type) {
    case 'SET_POSTS':
      return action.payload;
    case 'ADD_POST':
      return [...state, action.payload];
    default:
      return state;
  }
}

// Combined reducer
function rootReducer(state, action) {
  return {
    users: usersReducer(state.users, action),
    posts: postsReducer(state.posts, action),
  };
}

const initialState = {
  users: [],
  posts: [],
};

function App() {
  const [state, dispatch] = useReducer(rootReducer, initialState);
  // ...
}
```

## Reducer Factory

```jsx
function createCrudReducer(entityName) {
  const ACTIONS = {
    SET: `SET_${entityName.toUpperCase()}`,
    ADD: `ADD_${entityName.toUpperCase()}`,
    UPDATE: `UPDATE_${entityName.toUpperCase()}`,
    REMOVE: `REMOVE_${entityName.toUpperCase()}`,
  };

  function reducer(state = [], action) {
    switch (action.type) {
      case ACTIONS.SET:
        return action.payload;
      case ACTIONS.ADD:
        return [...state, action.payload];
      case ACTIONS.UPDATE:
        return state.map(item =>
          item.id === action.payload.id
            ? { ...item, ...action.payload }
            : item
        );
      case ACTIONS.REMOVE:
        return state.filter(item => item.id !== action.payload);
      default:
        return state;
    }
  }

  const actions = {
    set: (items) => ({ type: ACTIONS.SET, payload: items }),
    add: (item) => ({ type: ACTIONS.ADD, payload: item }),
    update: (item) => ({ type: ACTIONS.UPDATE, payload: item }),
    remove: (id) => ({ type: ACTIONS.REMOVE, payload: id }),
  };

  return { reducer, actions, ACTIONS };
}

// Usage
const usersReducerModule = createCrudReducer('users');
const postsReducerModule = createCrudReducer('posts');

function rootReducer(state, action) {
  return {
    users: usersReducerModule.reducer(state.users, action),
    posts: postsReducerModule.reducer(state.posts, action),
  };
}
```

## Middleware Pattern

```jsx
function useReducerWithMiddleware(reducer, initialState, middlewares = []) {
  const [state, dispatch] = useReducer(reducer, initialState);
  const stateRef = useRef(state);
  stateRef.current = state;

  const enhancedDispatch = useCallback((action) => {
    // Run middlewares
    middlewares.forEach(middleware => {
      middleware(stateRef.current, action);
    });
    dispatch(action);
  }, [middlewares]);

  return [state, enhancedDispatch];
}

// Logger middleware
const loggerMiddleware = (state, action) => {
  console.log('Action:', action.type);
  console.log('Current state:', state);
};

// Analytics middleware
const analyticsMiddleware = (state, action) => {
  if (action.type === 'PURCHASE') {
    analytics.track('purchase', action.payload);
  }
};

function App() {
  const [state, dispatch] = useReducerWithMiddleware(
    reducer,
    initialState,
    [loggerMiddleware, analyticsMiddleware]
  );
}
```

## Immer Integration

```jsx
import { useImmerReducer } from 'use-immer';

function todosReducer(draft, action) {
  switch (action.type) {
    case 'ADD':
      draft.push(action.payload);
      break;
    case 'TOGGLE':
      const todo = draft.find(t => t.id === action.payload);
      if (todo) {
        todo.completed = !todo.completed;
      }
      break;
    case 'REMOVE':
      const index = draft.findIndex(t => t.id === action.payload);
      if (index !== -1) {
        draft.splice(index, 1);
      }
      break;
  }
}

function TodoApp() {
  const [todos, dispatch] = useImmerReducer(todosReducer, []);
  // Mutations are safe with Immer!
}
```

---

> 📘 *This lesson is part of the [React State Architecture](https://stanza.dev/courses/react-state-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*