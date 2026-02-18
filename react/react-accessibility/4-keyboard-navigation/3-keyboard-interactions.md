---
source_course: "react-accessibility"
source_lesson: "react-accessibility-keyboard-interactions"
---

# Keyboard Interactions

Custom components need proper keyboard support.

## Expected Keyboard Patterns

### Buttons
- Enter/Space: Activate

```jsx
// Native button handles this automatically!
<button onClick={handleClick}>Click</button>
```

### Links
- Enter: Follow link

### Checkboxes
- Space: Toggle

### Radio Buttons
- Arrow keys: Navigate group
- Space: Select

### Tabs
- Arrow keys: Navigate tabs
- Tab: Move out of tab list

## Custom Keyboard Handler

```jsx
function CustomButton({ onClick, children }) {
  const handleKeyDown = (e) => {
    if (e.key === 'Enter' || e.key === ' ') {
      e.preventDefault();
      onClick();
    }
  };

  return (
    <div
      role="button"
      tabIndex={0}
      onClick={onClick}
      onKeyDown={handleKeyDown}
    >
      {children}
    </div>
  );
}
```

## Accessible Dropdown Menu

```jsx
function Dropdown({ label, items }) {
  const [isOpen, setIsOpen] = useState(false);
  const [activeIndex, setActiveIndex] = useState(-1);
  const buttonRef = useRef(null);
  const menuRef = useRef(null);

  const handleKeyDown = (e) => {
    switch (e.key) {
      case 'ArrowDown':
        e.preventDefault();
        if (!isOpen) {
          setIsOpen(true);
          setActiveIndex(0);
        } else {
          setActiveIndex(i => Math.min(i + 1, items.length - 1));
        }
        break;
      case 'ArrowUp':
        e.preventDefault();
        setActiveIndex(i => Math.max(i - 1, 0));
        break;
      case 'Escape':
        setIsOpen(false);
        buttonRef.current?.focus();
        break;
      case 'Enter':
      case ' ':
        if (isOpen && activeIndex >= 0) {
          e.preventDefault();
          items[activeIndex].onClick();
          setIsOpen(false);
        }
        break;
    }
  };

  return (
    <div onKeyDown={handleKeyDown}>
      <button
        ref={buttonRef}
        aria-haspopup="menu"
        aria-expanded={isOpen}
        onClick={() => setIsOpen(!isOpen)}
      >
        {label}
      </button>
      
      {isOpen && (
        <ul role="menu" ref={menuRef}>
          {items.map((item, index) => (
            <li
              key={item.id}
              role="menuitem"
              tabIndex={-1}
              aria-selected={index === activeIndex}
              className={index === activeIndex ? 'active' : ''}
              onClick={item.onClick}
            >
              {item.label}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

## Roving tabIndex

For grouped elements (tabs, toolbars):

```jsx
function Toolbar({ items }) {
  const [activeIndex, setActiveIndex] = useState(0);

  return (
    <div role="toolbar">
      {items.map((item, index) => (
        <button
          key={item.id}
          tabIndex={index === activeIndex ? 0 : -1}
          onClick={() => setActiveIndex(index)}
        >
          {item.label}
        </button>
      ))}
    </div>
  );
}
```

---

> 📘 *This lesson is part of the [React Accessibility (a11y)](https://stanza.dev/courses/react-accessibility) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*