---
source_course: "react-components-patterns"
source_lesson: "react-components-patterns-compound-advanced"
---

# Advanced Compound Component Patterns

Take compound components further with controlled/uncontrolled modes, validation, and accessibility.

## Controlled & Uncontrolled Modes

Support both controlled and uncontrolled usage:

```tsx
type TabsProps = {
  // Uncontrolled
  defaultTab?: string;
  // Controlled
  activeTab?: string;
  onTabChange?: (id: string) => void;
  children: ReactNode;
};

function Tabs({ 
  defaultTab, 
  activeTab: controlledTab, 
  onTabChange,
  children 
}: TabsProps) {
  const [internalTab, setInternalTab] = useState(defaultTab ?? '');
  
  // Determine if controlled
  const isControlled = controlledTab !== undefined;
  const activeTab = isControlled ? controlledTab : internalTab;
  
  const setActiveTab = (id: string) => {
    if (!isControlled) {
      setInternalTab(id);
    }
    onTabChange?.(id);
  };
  
  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
}

// Uncontrolled usage
<Tabs defaultTab="home">
  {/* ... */}
</Tabs>

// Controlled usage
const [tab, setTab] = useState('home');
<Tabs activeTab={tab} onTabChange={setTab}>
  {/* ... */}
</Tabs>
```

## Accordion Example

```tsx
type AccordionContextType = {
  openItems: Set<string>;
  toggle: (id: string) => void;
  allowMultiple: boolean;
};

const AccordionContext = createContext<AccordionContextType | null>(null);

type AccordionProps = {
  allowMultiple?: boolean;
  defaultOpen?: string[];
  children: ReactNode;
};

function Accordion({ 
  allowMultiple = false, 
  defaultOpen = [],
  children 
}: AccordionProps) {
  const [openItems, setOpenItems] = useState(new Set(defaultOpen));
  
  const toggle = (id: string) => {
    setOpenItems(prev => {
      const next = new Set(prev);
      
      if (next.has(id)) {
        next.delete(id);
      } else {
        if (!allowMultiple) {
          next.clear();
        }
        next.add(id);
      }
      
      return next;
    });
  };
  
  return (
    <AccordionContext.Provider value={{ openItems, toggle, allowMultiple }}>
      <div className="accordion">{children}</div>
    </AccordionContext.Provider>
  );
}

type AccordionItemProps = {
  id: string;
  children: ReactNode;
};

function AccordionItem({ id, children }: AccordionItemProps) {
  const { openItems } = useAccordionContext();
  const isOpen = openItems.has(id);
  
  return (
    <div className={`accordion-item ${isOpen ? 'open' : ''}`}>
      {children}
    </div>
  );
}

type AccordionTriggerProps = {
  id: string;
  children: ReactNode;
};

function AccordionTrigger({ id, children }: AccordionTriggerProps) {
  const { openItems, toggle } = useAccordionContext();
  const isOpen = openItems.has(id);
  
  return (
    <button
      className="accordion-trigger"
      aria-expanded={isOpen}
      aria-controls={`panel-${id}`}
      onClick={() => toggle(id)}
    >
      {children}
      <span className="icon">{isOpen ? '−' : '+'}</span>
    </button>
  );
}

type AccordionContentProps = {
  id: string;
  children: ReactNode;
};

function AccordionContent({ id, children }: AccordionContentProps) {
  const { openItems } = useAccordionContext();
  const isOpen = openItems.has(id);
  
  return (
    <div
      id={`panel-${id}`}
      className="accordion-content"
      hidden={!isOpen}
    >
      {children}
    </div>
  );
}

Accordion.Item = AccordionItem;
Accordion.Trigger = AccordionTrigger;
Accordion.Content = AccordionContent;
```

## Usage

```tsx
<Accordion allowMultiple defaultOpen={['faq-1']}>
  <Accordion.Item id="faq-1">
    <Accordion.Trigger id="faq-1">
      What is React?
    </Accordion.Trigger>
    <Accordion.Content id="faq-1">
      React is a JavaScript library for building user interfaces.
    </Accordion.Content>
  </Accordion.Item>
  
  <Accordion.Item id="faq-2">
    <Accordion.Trigger id="faq-2">
      Why use compound components?
    </Accordion.Trigger>
    <Accordion.Content id="faq-2">
      They provide flexible, composable APIs.
    </Accordion.Content>
  </Accordion.Item>
</Accordion>
```

## Resources

- [Controlled and Uncontrolled Components](https://react.dev/learn/sharing-state-between-components#controlled-and-uncontrolled-components) — React documentation on controlled vs uncontrolled patterns

---

> 📘 *This lesson is part of the [React Components & Patterns](https://stanza.dev/courses/react-components-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*