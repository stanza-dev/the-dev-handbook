---
source_course: "react-custom-hooks"
source_lesson: "react-custom-hooks-collection-hooks"
---

# useArray and useMap

Manage collection state with convenient helpers.

## useArray

```jsx
function useArray(initialValue = []) {
  const [array, setArray] = useState(initialValue);
  
  const push = useCallback((item) => {
    setArray(a => [...a, item]);
  }, []);
  
  const filter = useCallback((predicate) => {
    setArray(a => a.filter(predicate));
  }, []);
  
  const update = useCallback((index, newItem) => {
    setArray(a => a.map((item, i) => i === index ? newItem : item));
  }, []);
  
  const remove = useCallback((index) => {
    setArray(a => a.filter((_, i) => i !== index));
  }, []);
  
  const clear = useCallback(() => setArray([]), []);
  
  return {
    array,
    set: setArray,
    push,
    filter,
    update,
    remove,
    clear,
  };
}

// Usage
function TodoList() {
  const { array: todos, push, remove, update } = useArray([]);
  
  const addTodo = (text) => {
    push({ id: Date.now(), text, done: false });
  };
  
  const toggleTodo = (index) => {
    update(index, { ...todos[index], done: !todos[index].done });
  };
  
  return (
    <ul>
      {todos.map((todo, index) => (
        <li key={todo.id}>
          <input
            type="checkbox"
            checked={todo.done}
            onChange={() => toggleTodo(index)}
          />
          {todo.text}
          <button onClick={() => remove(index)}>×</button>
        </li>
      ))}
    </ul>
  );
}
```

## useMap (Object State)

```jsx
function useMap(initialValue = {}) {
  const [map, setMap] = useState(initialValue);
  
  const set = useCallback((key, value) => {
    setMap(m => ({ ...m, [key]: value }));
  }, []);
  
  const remove = useCallback((key) => {
    setMap(m => {
      const { [key]: _, ...rest } = m;
      return rest;
    });
  }, []);
  
  const has = useCallback((key) => key in map, [map]);
  
  const reset = useCallback(() => setMap(initialValue), [initialValue]);
  
  return {
    map,
    set,
    remove,
    has,
    reset,
    setAll: setMap,
  };
}

// Usage
function FormFields() {
  const { map: values, set } = useMap({ name: '', email: '' });
  
  return (
    <form>
      <input
        value={values.name}
        onChange={(e) => set('name', e.target.value)}
        placeholder="Name"
      />
      <input
        value={values.email}
        onChange={(e) => set('email', e.target.value)}
        placeholder="Email"
      />
    </form>
  );
}
```

## useSet

```jsx
function useSet(initialValue = new Set()) {
  const [set, setSet] = useState(initialValue);
  
  const add = useCallback((item) => {
    setSet(s => new Set([...s, item]));
  }, []);
  
  const remove = useCallback((item) => {
    setSet(s => {
      const newSet = new Set(s);
      newSet.delete(item);
      return newSet;
    });
  }, []);
  
  const toggle = useCallback((item) => {
    setSet(s => {
      const newSet = new Set(s);
      if (newSet.has(item)) {
        newSet.delete(item);
      } else {
        newSet.add(item);
      }
      return newSet;
    });
  }, []);
  
  const has = useCallback((item) => set.has(item), [set]);
  
  return {
    set,
    add,
    remove,
    toggle,
    has,
    clear: () => setSet(new Set()),
  };
}

// Usage - Selection state
function SelectableList({ items }) {
  const { set: selected, toggle, has } = useSet();
  
  return (
    <ul>
      {items.map(item => (
        <li
          key={item.id}
          onClick={() => toggle(item.id)}
          className={has(item.id) ? 'selected' : ''}
        >
          {item.name}
        </li>
      ))}
    </ul>
  );
}
```

---

> 📘 *This lesson is part of the [Building Custom Hooks](https://stanza.dev/courses/react-custom-hooks) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*