---
source_course: "react-modern-patterns"
source_lesson: "react-compound-components"
---

# Compound Components Pattern

## Introduction

The compound components pattern creates a set of components that work together to form a complete UI element while sharing implicit state. This pattern provides a flexible, declarative API that lets consumers control the arrangement and composition of sub-components.

## Key Concepts

- **Compound component**: A group of components that share implicit state through React Context, designed to be used together.
- **Implicit state sharing**: Child components access shared state without the parent explicitly passing props to each one.
- **Dot notation API**: Attaching sub-components as properties of the main component (e.g., `Tabs.Tab`, `Tabs.Panel`).

## Real World Context

HTML already uses compound components: `<select>` with `<option>`, `<table>` with `<tr>` and `<td>`. In React, this pattern is used by popular libraries: Radix UI's Dialog, Headless UI's Listbox, and Reach UI's Tabs. It gives consumers the flexibility to arrange, style, and extend components without the library needing to anticipate every use case.

## Deep Dive

The pattern uses Context to share state between a parent component and its children. The parent component provides the context, and child components consume it.

```tsx
import { createContext, useContext, useState, ReactNode } from 'react';

type TabsContextType = {
  activeTab: string;
  setActiveTab: (id: string) => void;
};

const TabsContext = createContext<TabsContextType | null>(null);

function useTabs() {
  const ctx = useContext(TabsContext);
  if (!ctx) throw new Error('Must be used within <Tabs>');
  return ctx;
}

function Tabs({ children, defaultTab }: { children: ReactNode; defaultTab: string }) {
  const [activeTab, setActiveTab] = useState(defaultTab);
  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
}

function Tab({ id, children }: { id: string; children: ReactNode }) {
  const { activeTab, setActiveTab } = useTabs();
  return (
    <button role="tab" aria-selected={activeTab === id} onClick={() => setActiveTab(id)}>
      {children}
    </button>
  );
}

function Panel({ id, children }: { id: string; children: ReactNode }) {
  const { activeTab } = useTabs();
  if (activeTab !== id) return null;
  return <div role="tabpanel">{children}</div>;
}

Tabs.Tab = Tab;
Tabs.Panel = Panel;

// Usage
<Tabs defaultTab="profile">
  <Tabs.Tab id="profile">Profile</Tabs.Tab>
  <Tabs.Tab id="settings">Settings</Tabs.Tab>
  <Tabs.Panel id="profile"><ProfileContent /></Tabs.Panel>
  <Tabs.Panel id="settings"><SettingsContent /></Tabs.Panel>
</Tabs>
```

The key benefits are: consumers control the layout and arrangement, state is shared without prop drilling, and the API is semantic and readable.

## Common Pitfalls

1. **Using compound components for simple use cases** — If the component only has one configuration, a simple props API is clearer. Compound components shine when flexibility is needed.
2. **Not validating context usage** — Always throw an error in the custom hook if the context is null, so developers get a clear message when using sub-components outside the parent.

## Best Practices

1. **Use Context for state sharing** — It is cleaner than cloneElement and works regardless of nesting depth.
2. **Provide clear error messages** — Throw descriptive errors when sub-components are used outside their parent provider.

## Summary

- Compound components share implicit state through Context for a flexible, declarative API.
- They follow the HTML pattern of related elements working together (select/option, table/tr/td).
- Use the dot notation pattern for a clean, discoverable API.

## Code Examples

**Compound component pattern for an Accordion**

```tsx
import { createContext, useContext, useState, ReactNode } from 'react';

type AccordionCtx = { openId: string | null; toggle: (id: string) => void };
const AccordionContext = createContext<AccordionCtx | null>(null);

function Accordion({ children }: { children: ReactNode }) {
  const [openId, setOpenId] = useState<string | null>(null);
  const toggle = (id: string) => setOpenId(prev => prev === id ? null : id);
  return (
    <AccordionContext.Provider value={{ openId, toggle }}>
      <div>{children}</div>
    </AccordionContext.Provider>
  );
}

function Item({ id, title, children }: { id: string; title: string; children: ReactNode }) {
  const ctx = useContext(AccordionContext);
  if (!ctx) throw new Error('Use within <Accordion>');
  return (
    <div>
      <button onClick={() => ctx.toggle(id)}>{title}</button>
      {ctx.openId === id && <div>{children}</div>}
    </div>
  );
}

Accordion.Item = Item;
```


## Resources

- [Passing Data Deeply with Context](https://react.dev/learn/passing-data-deeply-with-context) — Official React guide on using Context for deep data passing

---

> 📘 *This lesson is part of the [React 19 & Patterns](https://stanza.dev/courses/react-modern-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*