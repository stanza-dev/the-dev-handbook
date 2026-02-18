---
source_course: "react-custom-hooks"
source_lesson: "react-custom-hooks-toggle-boolean"
---

# useToggle and useBoolean

Simplify boolean state management.

## useToggle

```jsx
function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);
  
  const toggle = useCallback(() => setValue(v => !v), []);
  
  return [value, toggle];
}

// Usage
function Modal() {
  const [isOpen, toggleOpen] = useToggle(false);
  
  return (
    <>
      <button onClick={toggleOpen}>
        {isOpen ? 'Close' : 'Open'}
      </button>
      {isOpen && <ModalContent />}
    </>
  );
}
```

## useBoolean (Extended)

```jsx
function useBoolean(initialValue = false) {
  const [value, setValue] = useState(initialValue);
  
  const setTrue = useCallback(() => setValue(true), []);
  const setFalse = useCallback(() => setValue(false), []);
  const toggle = useCallback(() => setValue(v => !v), []);
  
  return {
    value,
    setValue,
    setTrue,
    setFalse,
    toggle,
  };
}

// Usage
function Dropdown() {
  const { value: isOpen, setTrue: open, setFalse: close, toggle } = useBoolean();
  
  return (
    <div>
      <button onClick={toggle}>Toggle</button>
      <button onClick={open}>Open</button>
      {isOpen && (
        <div onMouseLeave={close}>
          Dropdown content
        </div>
      )}
    </div>
  );
}
```

## TypeScript Version

```tsx
type UseBooleanReturn = {
  value: boolean;
  setValue: React.Dispatch<React.SetStateAction<boolean>>;
  setTrue: () => void;
  setFalse: () => void;
  toggle: () => void;
};

function useBoolean(initialValue = false): UseBooleanReturn {
  const [value, setValue] = useState(initialValue);
  
  const setTrue = useCallback(() => setValue(true), []);
  const setFalse = useCallback(() => setValue(false), []);
  const toggle = useCallback(() => setValue(v => !v), []);
  
  return {
    value,
    setValue,
    setTrue,
    setFalse,
    toggle,
  };
}
```

## useDisclosure (For Modals/Dialogs)

```tsx
function useDisclosure(initialOpen = false) {
  const { value: isOpen, setTrue: onOpen, setFalse: onClose, toggle: onToggle } = 
    useBoolean(initialOpen);
  
  // Escape key handler
  useEffect(() => {
    if (!isOpen) return;
    
    const handleEscape = (e: KeyboardEvent) => {
      if (e.key === 'Escape') onClose();
    };
    
    document.addEventListener('keydown', handleEscape);
    return () => document.removeEventListener('keydown', handleEscape);
  }, [isOpen, onClose]);
  
  return { isOpen, onOpen, onClose, onToggle };
}

// Usage
function App() {
  const { isOpen, onOpen, onClose } = useDisclosure();
  
  return (
    <>
      <button onClick={onOpen}>Open Modal</button>
      <Modal isOpen={isOpen} onClose={onClose}>
        <p>Modal content</p>
      </Modal>
    </>
  );
}
```

---

> 📘 *This lesson is part of the [Building Custom Hooks](https://stanza.dev/courses/react-custom-hooks) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*