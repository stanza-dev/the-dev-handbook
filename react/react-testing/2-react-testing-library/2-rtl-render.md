---
source_course: "react-testing"
source_lesson: "react-testing-rtl-render"
---

# Rendering Components

The `render` function is how you mount components for testing.

## Basic Rendering

```jsx
import { render, screen } from '@testing-library/react';
import Button from './Button';

test('renders button with text', () => {
  render(<Button>Click me</Button>);
  
  expect(screen.getByRole('button', { name: 'Click me' }))
    .toBeInTheDocument();
});
```

## Rendering with Props

```jsx
test('renders disabled button', () => {
  render(<Button disabled>Submit</Button>);
  
  const button = screen.getByRole('button');
  expect(button).toBeDisabled();
});

test('calls onClick when clicked', () => {
  const handleClick = jest.fn();
  render(<Button onClick={handleClick}>Click</Button>);
  
  fireEvent.click(screen.getByRole('button'));
  expect(handleClick).toHaveBeenCalledTimes(1);
});
```

## Rendering with Context

```jsx
import { ThemeProvider } from './ThemeContext';

function renderWithTheme(ui, theme = 'light') {
  return render(
    <ThemeProvider value={theme}>
      {ui}
    </ThemeProvider>
  );
}

test('renders in dark mode', () => {
  renderWithTheme(<Header />, 'dark');
  // ...
});
```

## Custom Render Function

Create a custom render with all providers:

```jsx
// test-utils.jsx
import { render } from '@testing-library/react';
import { BrowserRouter } from 'react-router-dom';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { AuthProvider } from './AuthContext';

function AllProviders({ children }) {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false }
    }
  });

  return (
    <QueryClientProvider client={queryClient}>
      <BrowserRouter>
        <AuthProvider>
          {children}
        </AuthProvider>
      </BrowserRouter>
    </QueryClientProvider>
  );
}

function customRender(ui, options) {
  return render(ui, { wrapper: AllProviders, ...options });
}

// Re-export everything
export * from '@testing-library/react';
export { customRender as render };
```

## Rerendering

```jsx
test('updates when props change', () => {
  const { rerender } = render(<Counter count={0} />);
  expect(screen.getByText('Count: 0')).toBeInTheDocument();
  
  rerender(<Counter count={5} />);
  expect(screen.getByText('Count: 5')).toBeInTheDocument();
});
```

## Cleanup

React Testing Library automatically cleans up after each test. You don't need to manually unmount.

---

> 📘 *This lesson is part of the [React Testing Strategies](https://stanza.dev/courses/react-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*