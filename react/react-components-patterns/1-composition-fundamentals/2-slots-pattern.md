---
source_course: "react-components-patterns"
source_lesson: "react-components-patterns-slots-pattern"
---

# The Slots Pattern: Multiple Content Areas

When you need multiple "holes" in a component, pass React elements as named props instead of using just `children`.

## Basic Slots

```tsx
type LayoutProps = {
  header: ReactNode;
  sidebar: ReactNode;
  children: ReactNode;
  footer?: ReactNode;
};

function Layout({ header, sidebar, children, footer }: LayoutProps) {
  return (
    <div className="layout">
      <header className="layout-header">{header}</header>
      <div className="layout-body">
        <aside className="layout-sidebar">{sidebar}</aside>
        <main className="layout-main">{children}</main>
      </div>
      {footer && <footer className="layout-footer">{footer}</footer>}
    </div>
  );
}

// Usage
<Layout
  header={<Navigation />}
  sidebar={<Menu items={menuItems} />}
  footer={<Copyright />}
>
  <Article content={content} />
</Layout>
```

## Dialog with Slots

```tsx
type DialogProps = {
  title: ReactNode;
  children: ReactNode;
  actions?: ReactNode;
  icon?: ReactNode;
};

function Dialog({ title, children, actions, icon }: DialogProps) {
  return (
    <div className="dialog-overlay">
      <div className="dialog">
        <div className="dialog-header">
          {icon && <span className="dialog-icon">{icon}</span>}
          <h2 className="dialog-title">{title}</h2>
        </div>
        <div className="dialog-body">
          {children}
        </div>
        {actions && (
          <div className="dialog-actions">
            {actions}
          </div>
        )}
      </div>
    </div>
  );
}

// Usage
<Dialog
  icon={<WarningIcon />}
  title="Delete Item?"
  actions={
    <>
      <Button variant="secondary" onClick={onCancel}>Cancel</Button>
      <Button variant="danger" onClick={onDelete}>Delete</Button>
    </>
  }
>
  <p>This action cannot be undone.</p>
</Dialog>
```

## Conditional Slots

```tsx
function Card({ 
  header, 
  children, 
  footer,
  media 
}: CardProps) {
  return (
    <div className="card">
      {media && (
        <div className="card-media">{media}</div>
      )}
      {header && (
        <div className="card-header">{header}</div>
      )}
      <div className="card-body">{children}</div>
      {footer && (
        <div className="card-footer">{footer}</div>
      )}
    </div>
  );
}

// Minimal usage
<Card>Just content</Card>

// Full usage
<Card
  media={<img src="/hero.jpg" alt="Hero" />}
  header={<h3>Card Title</h3>}
  footer={<Button>Action</Button>}
>
  <p>Card content goes here.</p>
</Card>
```

## Slots vs Children

| Aspect | children | Named Slots |
|--------|----------|-------------|
| Single content area | ✅ Perfect | Overkill |
| Multiple areas | ❌ Awkward | ✅ Clean |
| Optional areas | ❌ Hard | ✅ Easy |
| Semantic clarity | ❌ Generic | ✅ Clear intent |

## Advanced: Slot Validation

```tsx
function validateSlot(slot: ReactNode, name: string) {
  if (slot === undefined) return;
  
  if (!isValidElement(slot)) {
    console.warn(`${name} should be a React element`);
  }
}

function Layout({ header, sidebar, children }: LayoutProps) {
  if (process.env.NODE_ENV === 'development') {
    validateSlot(header, 'header');
    validateSlot(sidebar, 'sidebar');
  }
  
  return (/* ... */);
}
```

## Resources

- [Composition vs Inheritance](https://react.dev/learn/thinking-in-react) — React's approach to component composition

---

> 📘 *This lesson is part of the [React Components & Patterns](https://stanza.dev/courses/react-components-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*