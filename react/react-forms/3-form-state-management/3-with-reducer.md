---
source_course: "react-forms"
source_lesson: "react-forms-form-state-with-reducer"
---

# Form State with useReducer

For complex forms with many state transitions, `useReducer` provides better organization than `useState`.

## When to Use useReducer

- Multiple related state values
- Complex state logic
- Actions that depend on previous state
- When you want to centralize state logic

## Form Reducer Pattern

```jsx
import { useReducer } from 'react';

const initialState = {
  values: { email: '', password: '' },
  errors: {},
  touched: {},
  isSubmitting: false
};

function formReducer(state, action) {
  switch (action.type) {
    case 'SET_FIELD':
      return {
        ...state,
        values: {
          ...state.values,
          [action.field]: action.value
        }
      };
    
    case 'SET_ERROR':
      return {
        ...state,
        errors: {
          ...state.errors,
          [action.field]: action.error
        }
      };
    
    case 'SET_TOUCHED':
      return {
        ...state,
        touched: {
          ...state.touched,
          [action.field]: true
        }
      };
    
    case 'SUBMIT_START':
      return { ...state, isSubmitting: true };
    
    case 'SUBMIT_SUCCESS':
      return { ...initialState };
    
    case 'SUBMIT_ERROR':
      return {
        ...state,
        isSubmitting: false,
        errors: action.errors
      };
    
    case 'RESET':
      return initialState;
    
    default:
      return state;
  }
}

function LoginForm() {
  const [state, dispatch] = useReducer(formReducer, initialState);

  function handleChange(e) {
    dispatch({
      type: 'SET_FIELD',
      field: e.target.name,
      value: e.target.value
    });
  }

  function handleBlur(e) {
    dispatch({
      type: 'SET_TOUCHED',
      field: e.target.name
    });
  }

  async function handleSubmit(e) {
    e.preventDefault();
    dispatch({ type: 'SUBMIT_START' });
    
    try {
      await submitLogin(state.values);
      dispatch({ type: 'SUBMIT_SUCCESS' });
    } catch (err) {
      dispatch({ type: 'SUBMIT_ERROR', errors: err.errors });
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="email"
        value={state.values.email}
        onChange={handleChange}
        onBlur={handleBlur}
      />
      {state.touched.email && state.errors.email && (
        <span>{state.errors.email}</span>
      )}
      <button disabled={state.isSubmitting}>
        {state.isSubmitting ? 'Logging in...' : 'Log In'}
      </button>
    </form>
  );
}
```

📚 **Learn more**: [Extracting State Logic into a Reducer](https://react.dev/learn/extracting-state-logic-into-a-reducer)

---

> 📘 *This lesson is part of the [React Form Mastery](https://stanza.dev/courses/react-forms) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*