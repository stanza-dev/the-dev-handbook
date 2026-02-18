---
source_course: "react-custom-hooks"
source_lesson: "react-custom-hooks-event-hooks"
---

# useEventListener and useKeyPress

Simplify event handling.

## useEventListener

```jsx
function useEventListener(eventName, handler, element = window) {
  // Create a ref that stores handler
  const savedHandler = useRef();
  
  // Update ref.current if handler changes
  useEffect(() => {
    savedHandler.current = handler;
  }, [handler]);
  
  useEffect(() => {
    // Make sure element supports addEventListener
    const targetElement = element?.current || element;
    if (!(targetElement && targetElement.addEventListener)) return;
    
    // Create event listener that calls handler function stored in ref
    const eventListener = (event) => savedHandler.current(event);
    
    targetElement.addEventListener(eventName, eventListener);
    
    return () => {
      targetElement.removeEventListener(eventName, eventListener);
    };
  }, [eventName, element]);
}

// Usage
function App() {
  const [coords, setCoords] = useState({ x: 0, y: 0 });
  
  useEventListener('mousemove', (e) => {
    setCoords({ x: e.clientX, y: e.clientY });
  });
  
  return (
    <div>
      Mouse: {coords.x}, {coords.y}
    </div>
  );
}

// With element ref
function ScrollableBox() {
  const boxRef = useRef(null);
  const [scrollTop, setScrollTop] = useState(0);
  
  useEventListener('scroll', (e) => {
    setScrollTop(e.target.scrollTop);
  }, boxRef);
  
  return (
    <div ref={boxRef} style={{ overflow: 'auto', height: 200 }}>
      Scrolled: {scrollTop}px
      {/* content */}
    </div>
  );
}
```

## useKeyPress

```jsx
function useKeyPress(targetKey) {
  const [isPressed, setIsPressed] = useState(false);
  
  useEffect(() => {
    const downHandler = (e) => {
      if (e.key === targetKey) {
        setIsPressed(true);
      }
    };
    
    const upHandler = (e) => {
      if (e.key === targetKey) {
        setIsPressed(false);
      }
    };
    
    window.addEventListener('keydown', downHandler);
    window.addEventListener('keyup', upHandler);
    
    return () => {
      window.removeEventListener('keydown', downHandler);
      window.removeEventListener('keyup', upHandler);
    };
  }, [targetKey]);
  
  return isPressed;
}

// Usage
function Game() {
  const isUpPressed = useKeyPress('ArrowUp');
  const isDownPressed = useKeyPress('ArrowDown');
  const isSpacePressed = useKeyPress(' ');
  
  return (
    <div>
      {isUpPressed && <span>⬆️</span>}
      {isDownPressed && <span>⬇️</span>}
      {isSpacePressed && <span>🚀 Jump!</span>}
    </div>
  );
}
```

## useKeyboardShortcut

```jsx
function useKeyboardShortcut(keys, callback, options = {}) {
  const { ctrl = false, alt = false, shift = false } = options;
  
  useEffect(() => {
    const handleKeyDown = (event) => {
      const isCtrlMatch = ctrl === (event.ctrlKey || event.metaKey);
      const isAltMatch = alt === event.altKey;
      const isShiftMatch = shift === event.shiftKey;
      
      const targetKeys = Array.isArray(keys) ? keys : [keys];
      const isKeyMatch = targetKeys.includes(event.key.toLowerCase());
      
      if (isCtrlMatch && isAltMatch && isShiftMatch && isKeyMatch) {
        event.preventDefault();
        callback(event);
      }
    };
    
    document.addEventListener('keydown', handleKeyDown);
    return () => document.removeEventListener('keydown', handleKeyDown);
  }, [keys, callback, ctrl, alt, shift]);
}

// Usage
function Editor() {
  useKeyboardShortcut('s', () => save(), { ctrl: true });
  useKeyboardShortcut('z', () => undo(), { ctrl: true });
  useKeyboardShortcut('z', () => redo(), { ctrl: true, shift: true });
  useKeyboardShortcut('escape', () => closeModal());
  
  return <TextEditor />;
}
```

## useClickOutside

```jsx
function useClickOutside(ref, handler) {
  useEffect(() => {
    const listener = (event) => {
      if (!ref.current || ref.current.contains(event.target)) {
        return;
      }
      handler(event);
    };
    
    document.addEventListener('mousedown', listener);
    document.addEventListener('touchstart', listener);
    
    return () => {
      document.removeEventListener('mousedown', listener);
      document.removeEventListener('touchstart', listener);
    };
  }, [ref, handler]);
}

// Usage
function Dropdown() {
  const ref = useRef(null);
  const [isOpen, setIsOpen] = useState(false);
  
  useClickOutside(ref, () => setIsOpen(false));
  
  return (
    <div ref={ref}>
      <button onClick={() => setIsOpen(!isOpen)}>Toggle</button>
      {isOpen && <DropdownMenu />}
    </div>
  );
}
```

---

> 📘 *This lesson is part of the [Building Custom Hooks](https://stanza.dev/courses/react-custom-hooks) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*