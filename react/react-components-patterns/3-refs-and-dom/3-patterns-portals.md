---
source_course: "react-components-patterns"
source_lesson: "react-components-patterns-portals"
---

# Portals: Rendering Outside the Parent

Portals let you render children into a DOM node that exists outside the parent component's DOM hierarchy.

## Why Portals?

Some UI elements need to visually "break out" of their container:

- **Modals** - Should overlay the entire page
- **Tooltips** - Need to escape `overflow: hidden`
- **Dropdowns** - Must not be clipped by parent containers
- **Toasts** - Float above all content

## Basic Usage

```tsx
import { createPortal } from 'react-dom';

function Modal({ isOpen, onClose, children }) {
  if (!isOpen) return null;
  
  return createPortal(
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal" onClick={e => e.stopPropagation()}>
        {children}
      </div>
    </div>,
    document.body
  );
}
```

## Complete Modal Implementation

```tsx
import { createPortal } from 'react-dom';
import { useEffect, useRef } from 'react';

type ModalProps = {
  isOpen: boolean;
  onClose: () => void;
  title: string;
  children: ReactNode;
};

function Modal({ isOpen, onClose, title, children }: ModalProps) {
  const previousActiveElement = useRef<Element | null>(null);
  const modalRef = useRef<HTMLDivElement>(null);
  
  useEffect(() => {
    if (isOpen) {
      // Save current focus
      previousActiveElement.current = document.activeElement;
      
      // Focus the modal
      modalRef.current?.focus();
      
      // Prevent body scroll
      document.body.style.overflow = 'hidden';
      
      // Handle escape key
      const handleEscape = (e: KeyboardEvent) => {
        if (e.key === 'Escape') onClose();
      };
      document.addEventListener('keydown', handleEscape);
      
      return () => {
        document.removeEventListener('keydown', handleEscape);
        document.body.style.overflow = '';
        
        // Restore focus
        (previousActiveElement.current as HTMLElement)?.focus();
      };
    }
  }, [isOpen, onClose]);
  
  if (!isOpen) return null;
  
  return createPortal(
    <div 
      className="modal-overlay" 
      onClick={onClose}
      role="presentation"
    >
      <div
        ref={modalRef}
        className="modal"
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-title"
        tabIndex={-1}
        onClick={e => e.stopPropagation()}
      >
        <header className="modal-header">
          <h2 id="modal-title">{title}</h2>
          <button 
            onClick={onClose}
            aria-label="Close modal"
          >
            ×
          </button>
        </header>
        <div className="modal-body">
          {children}
        </div>
      </div>
    </div>,
    document.body
  );
}
```

## Event Bubbling

Even though the DOM is elsewhere, events bubble through the React tree:

```tsx
function Parent() {
  const handleClick = () => console.log('Parent clicked!');
  
  return (
    <div onClick={handleClick}>
      <Modal isOpen={true} onClose={() => {}}>
        <button>Click me</button>
        {/* Click bubbles to Parent in React tree! */}
      </Modal>
    </div>
  );
}
```

## Tooltip with Portal

```tsx
function Tooltip({ children, content, targetRef }) {
  const [position, setPosition] = useState({ top: 0, left: 0 });
  const [isVisible, setIsVisible] = useState(false);
  
  useLayoutEffect(() => {
    if (isVisible && targetRef.current) {
      const rect = targetRef.current.getBoundingClientRect();
      setPosition({
        top: rect.bottom + window.scrollY + 8,
        left: rect.left + window.scrollX + rect.width / 2
      });
    }
  }, [isVisible, targetRef]);
  
  return (
    <>
      <span
        ref={targetRef}
        onMouseEnter={() => setIsVisible(true)}
        onMouseLeave={() => setIsVisible(false)}
      >
        {children}
      </span>
      {isVisible && createPortal(
        <div
          className="tooltip"
          style={{ top: position.top, left: position.left }}
        >
          {content}
        </div>,
        document.body
      )}
    </>
  );
}
```

## Resources

- [createPortal](https://react.dev/reference/react-dom/createPortal) — Official React documentation for createPortal

---

> 📘 *This lesson is part of the [React Components & Patterns](https://stanza.dev/courses/react-components-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*