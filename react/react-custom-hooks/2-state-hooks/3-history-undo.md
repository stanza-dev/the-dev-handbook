---
source_course: "react-custom-hooks"
source_lesson: "react-custom-hooks-history-undo"
---

# useHistory for Undo/Redo

Implement undo/redo functionality.

## Basic useHistory

```jsx
function useHistory(initialState) {
  const [index, setIndex] = useState(0);
  const [history, setHistory] = useState([initialState]);
  
  const state = history[index];
  
  const setState = useCallback((newState) => {
    const resolvedState = typeof newState === 'function' 
      ? newState(history[index]) 
      : newState;
    
    // Discard any "future" history
    const newHistory = history.slice(0, index + 1);
    setHistory([...newHistory, resolvedState]);
    setIndex(newHistory.length);
  }, [history, index]);
  
  const undo = useCallback(() => {
    setIndex(i => Math.max(0, i - 1));
  }, []);
  
  const redo = useCallback(() => {
    setIndex(i => Math.min(history.length - 1, i + 1));
  }, [history.length]);
  
  const canUndo = index > 0;
  const canRedo = index < history.length - 1;
  
  return {
    state,
    setState,
    undo,
    redo,
    canUndo,
    canRedo,
    history,
    index,
  };
}

// Usage
function TextEditor() {
  const {
    state: text,
    setState: setText,
    undo,
    redo,
    canUndo,
    canRedo,
  } = useHistory('');
  
  return (
    <div>
      <div>
        <button onClick={undo} disabled={!canUndo}>Undo</button>
        <button onClick={redo} disabled={!canRedo}>Redo</button>
      </div>
      <textarea
        value={text}
        onChange={(e) => setText(e.target.value)}
      />
    </div>
  );
}
```

## With Max History Limit

```jsx
function useHistory(initialState, { maxLength = 100 } = {}) {
  const [index, setIndex] = useState(0);
  const [history, setHistory] = useState([initialState]);
  
  const state = history[index];
  
  const setState = useCallback((newState) => {
    const resolvedState = typeof newState === 'function' 
      ? newState(history[index]) 
      : newState;
    
    setHistory(prevHistory => {
      // Discard future
      let newHistory = prevHistory.slice(0, index + 1);
      
      // Add new state
      newHistory = [...newHistory, resolvedState];
      
      // Trim if exceeds max
      if (newHistory.length > maxLength) {
        newHistory = newHistory.slice(-maxLength);
        setIndex(newHistory.length - 1);
        return newHistory;
      }
      
      setIndex(newHistory.length - 1);
      return newHistory;
    });
  }, [history, index, maxLength]);
  
  // ... rest same as before
}
```

## useUndo with Reset

```jsx
function useUndo(initialState) {
  const {
    state,
    setState,
    undo,
    redo,
    canUndo,
    canRedo,
  } = useHistory(initialState);
  
  const reset = useCallback((newInitialState = initialState) => {
    setState(newInitialState);
  }, [initialState, setState]);
  
  return [
    state,
    {
      set: setState,
      undo,
      redo,
      reset,
      canUndo,
      canRedo,
    },
  ];
}

// Usage
function DrawingCanvas() {
  const [paths, { set, undo, redo, reset, canUndo, canRedo }] = useUndo([]);
  
  const addPath = (path) => {
    set(current => [...current, path]);
  };
  
  return (
    <div>
      <button onClick={undo} disabled={!canUndo}>⟲ Undo</button>
      <button onClick={redo} disabled={!canRedo}>⟳ Redo</button>
      <button onClick={() => reset([])}>🗑 Clear</button>
      <Canvas paths={paths} onDraw={addPath} />
    </div>
  );
}
```

---

> 📘 *This lesson is part of the [Building Custom Hooks](https://stanza.dev/courses/react-custom-hooks) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*