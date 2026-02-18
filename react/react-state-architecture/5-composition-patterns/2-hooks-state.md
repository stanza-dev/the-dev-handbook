---
source_course: "react-state-architecture"
source_lesson: "react-state-architecture-custom-hooks-state"
---

# Custom Hooks for State Logic

Encapsulate reusable state patterns in hooks.

## useToggle

```jsx
function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);
  
  const toggle = useCallback(() => setValue(v => !v), []);
  const setTrue = useCallback(() => setValue(true), []);
  const setFalse = useCallback(() => setValue(false), []);
  
  return [value, { toggle, setTrue, setFalse }];
}

// Usage
function Modal() {
  const [isOpen, { toggle, setFalse }] = useToggle();
  
  return (
    <>
      <button onClick={toggle}>Open</button>
      {isOpen && <Dialog onClose={setFalse} />}
    </>
  );
}
```

## useList

```jsx
function useList(initialList = []) {
  const [list, setList] = useState(initialList);
  
  const push = useCallback((item) => {
    setList(l => [...l, item]);
  }, []);
  
  const removeAt = useCallback((index) => {
    setList(l => l.filter((_, i) => i !== index));
  }, []);
  
  const removeById = useCallback((id) => {
    setList(l => l.filter(item => item.id !== id));
  }, []);
  
  const updateAt = useCallback((index, newItem) => {
    setList(l => l.map((item, i) => i === index ? newItem : item));
  }, []);
  
  const clear = useCallback(() => setList([]), []);
  
  return [list, { push, removeAt, removeById, updateAt, clear, set: setList }];
}

// Usage
function TodoList() {
  const [todos, { push, removeById }] = useList([]);
  
  const addTodo = (text) => {
    push({ id: Date.now(), text, completed: false });
  };
  
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          {todo.text}
          <button onClick={() => removeById(todo.id)}>×</button>
        </li>
      ))}
    </ul>
  );
}
```

## useMap (Object State)

```jsx
function useMap(initialMap = {}) {
  const [map, setMap] = useState(initialMap);
  
  const set = useCallback((key, value) => {
    setMap(m => ({ ...m, [key]: value }));
  }, []);
  
  const remove = useCallback((key) => {
    setMap(m => {
      const { [key]: _, ...rest } = m;
      return rest;
    });
  }, []);
  
  const reset = useCallback(() => setMap(initialMap), [initialMap]);
  
  return [map, { set, remove, reset, setAll: setMap }];
}
```

## useHistory (Undo/Redo)

```jsx
function useHistory(initialState) {
  const [history, setHistory] = useState({
    past: [],
    present: initialState,
    future: [],
  });
  
  const canUndo = history.past.length > 0;
  const canRedo = history.future.length > 0;
  
  const set = useCallback((newState) => {
    setHistory(h => ({
      past: [...h.past, h.present],
      present: typeof newState === 'function' ? newState(h.present) : newState,
      future: [],
    }));
  }, []);
  
  const undo = useCallback(() => {
    setHistory(h => {
      if (h.past.length === 0) return h;
      const previous = h.past[h.past.length - 1];
      const newPast = h.past.slice(0, -1);
      return {
        past: newPast,
        present: previous,
        future: [h.present, ...h.future],
      };
    });
  }, []);
  
  const redo = useCallback(() => {
    setHistory(h => {
      if (h.future.length === 0) return h;
      const next = h.future[0];
      const newFuture = h.future.slice(1);
      return {
        past: [...h.past, h.present],
        present: next,
        future: newFuture,
      };
    });
  }, []);
  
  return [history.present, { set, undo, redo, canUndo, canRedo }];
}

// Usage
function Editor() {
  const [text, { set, undo, redo, canUndo, canRedo }] = useHistory('');
  
  return (
    <div>
      <textarea value={text} onChange={e => set(e.target.value)} />
      <button onClick={undo} disabled={!canUndo}>Undo</button>
      <button onClick={redo} disabled={!canRedo}>Redo</button>
    </div>
  );
}
```

---

> 📘 *This lesson is part of the [React State Architecture](https://stanza.dev/courses/react-state-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*