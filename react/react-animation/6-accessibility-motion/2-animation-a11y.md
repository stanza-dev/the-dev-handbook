---
source_course: "react-animation"
source_lesson: "react-animation-focus-animation-a11y"
---

# Animation and Focus Management

Ensure animations don't interfere with keyboard navigation.

## Focus Visibility During Animations

```jsx
function Modal({ isOpen, onClose }) {
  const closeButtonRef = useRef(null);

  // Focus after animation completes
  useEffect(() => {
    if (isOpen) {
      // Wait for enter animation
      const timer = setTimeout(() => {
        closeButtonRef.current?.focus();
      }, 300); // Match animation duration
      return () => clearTimeout(timer);
    }
  }, [isOpen]);

  return (
    <AnimatePresence>
      {isOpen && (
        <motion.div
          initial={{ opacity: 0 }}
          animate={{ opacity: 1 }}
          exit={{ opacity: 0 }}
          transition={{ duration: 0.3 }}
        >
          <button ref={closeButtonRef}>Close</button>
        </motion.div>
      )}
    </AnimatePresence>
  );
}
```

## With Framer Motion's onAnimationComplete

```jsx
function AccessibleModal({ isOpen, onClose }) {
  const closeButtonRef = useRef(null);
  const previousFocusRef = useRef(null);

  useEffect(() => {
    if (isOpen) {
      previousFocusRef.current = document.activeElement;
    }
  }, [isOpen]);

  return (
    <AnimatePresence
      onExitComplete={() => {
        // Return focus after exit animation
        previousFocusRef.current?.focus();
      }}
    >
      {isOpen && (
        <motion.div
          initial={{ opacity: 0, scale: 0.95 }}
          animate={{ opacity: 1, scale: 1 }}
          exit={{ opacity: 0, scale: 0.95 }}
          onAnimationComplete={(definition) => {
            // Focus after enter animation
            if (definition === 'animate') {
              closeButtonRef.current?.focus();
            }
          }}
          className="modal"
        >
          <button ref={closeButtonRef}>Close</button>
        </motion.div>
      )}
    </AnimatePresence>
  );
}
```

## Animated Lists and Focus

```jsx
function AnimatedList({ items, onRemove }) {
  const listRef = useRef(null);
  const prevItemCount = useRef(items.length);

  useEffect(() => {
    // Focus list when item is removed
    if (items.length < prevItemCount.current) {
      listRef.current?.focus();
    }
    prevItemCount.current = items.length;
  }, [items.length]);

  return (
    <ul
      ref={listRef}
      tabIndex={-1}
      aria-live="polite"
    >
      <AnimatePresence>
        {items.map((item) => (
          <motion.li
            key={item.id}
            exit={{ opacity: 0 }}
            transition={{ duration: 0.2 }}
          >
            {item.name}
            <button onClick={() => onRemove(item.id)}>
              Remove
            </button>
          </motion.li>
        ))}
      </AnimatePresence>
    </ul>
  );
}
```

## Screen Reader Announcements

```jsx
function AnimatedNotification({ message, onDismiss }) {
  return (
    <motion.div
      role="alert"
      aria-live="assertive"
      initial={{ opacity: 0, y: -20 }}
      animate={{ opacity: 1, y: 0 }}
      exit={{ opacity: 0, y: -20 }}
    >
      {message}
      <button onClick={onDismiss} aria-label="Dismiss notification">
        ×
      </button>
    </motion.div>
  );
}
```

## Animation and Skip Links

```jsx
function PageTransition({ children }) {
  const mainRef = useRef(null);

  return (
    <>
      <a href="#main" className="skip-link">
        Skip to main content
      </a>
      <motion.main
        ref={mainRef}
        id="main"
        tabIndex={-1}
        initial={{ opacity: 0 }}
        animate={{ opacity: 1 }}
        onAnimationComplete={() => {
          // Focus main after page transition
          mainRef.current?.focus();
        }}
      >
        {children}
      </motion.main>
    </>
  );
}
```

---

> 📘 *This lesson is part of the [React Animation & Motion](https://stanza.dev/courses/react-animation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*