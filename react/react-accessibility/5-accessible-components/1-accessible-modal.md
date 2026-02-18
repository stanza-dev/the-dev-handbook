---
source_course: "react-accessibility"
source_lesson: "react-accessibility-accessible-modal"
---

# Accessible Modal

Modals require careful attention to accessibility.

## Requirements

1. Focus moves to modal on open
2. Focus is trapped inside
3. Escape closes modal
4. Focus returns to trigger on close
5. Background content is inert
6. Proper ARIA attributes

## Complete Modal Component

```jsx
import { useEffect, useRef, useCallback } from 'react';
import { createPortal } from 'react-dom';

function Modal({ isOpen, onClose, title, children }) {
  const modalRef = useRef(null);
  const previousFocusRef = useRef(null);

  // Handle escape key
  const handleKeyDown = useCallback((e) => {
    if (e.key === 'Escape') {
      onClose();
    }
  }, [onClose]);

  // Focus management
  useEffect(() => {
    if (isOpen) {
      previousFocusRef.current = document.activeElement;
      modalRef.current?.focus();
      document.addEventListener('keydown', handleKeyDown);
      document.body.style.overflow = 'hidden';
    }

    return () => {
      document.removeEventListener('keydown', handleKeyDown);
      document.body.style.overflow = '';
      if (isOpen) {
        previousFocusRef.current?.focus();
      }
    };
  }, [isOpen, handleKeyDown]);

  if (!isOpen) return null;

  return createPortal(
    <div className="modal-overlay" onClick={onClose}>
      <div
        ref={modalRef}
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-title"
        tabIndex={-1}
        className="modal"
        onClick={(e) => e.stopPropagation()}
      >
        <header>
          <h2 id="modal-title">{title}</h2>
          <button
            onClick={onClose}
            aria-label="Close dialog"
          >
            ×
          </button>
        </header>
        <div className="modal-content">
          {children}
        </div>
      </div>
    </div>,
    document.body
  );
}
```

## Usage

```jsx
function App() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <>
      <button onClick={() => setIsOpen(true)}>
        Open Modal
      </button>
      
      <Modal
        isOpen={isOpen}
        onClose={() => setIsOpen(false)}
        title="Confirm Action"
      >
        <p>Are you sure you want to proceed?</p>
        <button onClick={() => setIsOpen(false)}>
          Confirm
        </button>
      </Modal>
    </>
  );
}
```

## Key ARIA Attributes

```jsx
<div
  role="dialog"         // Identifies as dialog
  aria-modal="true"     // Indicates modal behavior
  aria-labelledby="id" // Points to title
  aria-describedby="id" // Points to description (optional)
>
```

---

> 📘 *This lesson is part of the [React Accessibility (a11y)](https://stanza.dev/courses/react-accessibility) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*