---
source_course: "nextjs-testing"
source_lesson: "nextjs-testing-njs-testing-client-components"
---

# Testing Client Components

Client Components use hooks and browser APIs, requiring a simulated DOM environment.

## Basic Test

```typescript
// components/Counter.tsx
'use client';
import { useState } from 'react';

export function Counter() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
    </div>
  );
}
```

```typescript
// components/Counter.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { Counter } from './Counter';

describe('Counter', () => {
  it('increments the count when button is clicked', () => {
    render(<Counter />);

    expect(screen.getByText('Count: 0')).toBeInTheDocument();

    fireEvent.click(screen.getByRole('button', { name: /increment/i }));

    expect(screen.getByText('Count: 1')).toBeInTheDocument();
  });
});
```

## Testing User Interactions

Use `@testing-library/user-event` for realistic interactions:

```typescript
import userEvent from '@testing-library/user-event';

it('handles typing', async () => {
  const user = userEvent.setup();
  render(<SearchInput />);

  await user.type(screen.getByRole('textbox'), 'hello');

  expect(screen.getByRole('textbox')).toHaveValue('hello');
});
```

---

> 📘 *This lesson is part of the [Next.js Testing Strategies](https://stanza.dev/courses/nextjs-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*