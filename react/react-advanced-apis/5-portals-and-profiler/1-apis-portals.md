---
source_course: "react-advanced-apis"
source_lesson: "react-advanced-apis-portals"
---

# createPortal: Rendering Outside the Parent

`createPortal` lets you render children into a different part of the DOM tree, outside the parent component's DOM hierarchy.

## Why Portals?

Some UI elements need to visually "break out" of their container:

- **Modals** - Should overlay the entire page
- **Tooltips** - Need to escape `overflow: hidden`
- **Dropdowns** - Must not be clipped by parent containers
- **Notifications** - Float above all content

## Basic Usage

```tsx
import { createPortal } from 'react-dom';

function Modal({ children, isOpen }) {
  if (!isOpen) return null;
  
  return createPortal(
    <div className="modal-overlay">
      <div className="modal-content">
        {children}
      </div>
    </div>,
    document.body // Render into body, not parent
  );
}
```

## Portal Target Setup

```html
<!-- index.html -->
<body>
  <div id="root"></div>
  <div id="modal-root"></div>
  <div id="tooltip-root"></div>
</body>
```

```tsx
function Modal({ children }) {
  return createPortal(
    children,
    document.getElementById('modal-root')
  );
}
```

## Event Bubbling Through Portals

Even though the DOM is elsewhere, events bubble through the React tree:

```tsx
function Parent() {
  const handleClick = () => console.log('Parent clicked!');
  
  return (
    <div onClick={handleClick}>
      <Modal>
        <button>Click me</button>
        {/* Click bubbles to Parent, even though Modal renders in body! */}
      </Modal>
    </div>
  );
}
```

## Complete Modal Example

```tsx
function Modal({ isOpen, onClose, children }) {
  useEffect(() => {
    const handleEscape = (e) => {
      if (e.key === 'Escape') onClose();
    };
    
    if (isOpen) {
      document.addEventListener('keydown', handleEscape);
      document.body.style.overflow = 'hidden';
    }
    
    return () => {
      document.removeEventListener('keydown', handleEscape);
      document.body.style.overflow = '';
    };
  }, [isOpen, onClose]);
  
  if (!isOpen) return null;
  
  return createPortal(
    <div 
      className="modal-overlay" 
      onClick={onClose}
      role="dialog"
      aria-modal="true"
    >
      <div 
        className="modal-content" 
        onClick={(e) => e.stopPropagation()}
      >
        {children}
      </div>
    </div>,
    document.getElementById('modal-root')
  );
}
```

## Tooltip with Portal

```tsx
function Tooltip({ targetRef, children }) {
  const [position, setPosition] = useState({ top: 0, left: 0 });
  
  useLayoutEffect(() => {
    if (targetRef.current) {
      const rect = targetRef.current.getBoundingClientRect();
      setPosition({
        top: rect.bottom + window.scrollY + 8,
        left: rect.left + window.scrollX
      });
    }
  }, [targetRef]);
  
  return createPortal(
    <div 
      className="tooltip" 
      style={{ position: 'absolute', ...position }}
    >
      {children}
    </div>,
    document.body
  );
}
```

## Resources

- [createPortal API Reference](https://react.dev/reference/react-dom/createPortal) — Official React documentation for createPortal

---

> 📘 *This lesson is part of the [React Advanced APIs](https://stanza.dev/courses/react-advanced-apis) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*