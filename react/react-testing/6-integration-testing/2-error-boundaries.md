---
source_course: "react-testing"
source_lesson: "react-testing-testing-error-boundaries"
---

# Testing Error Boundaries

Error boundaries catch JavaScript errors and display fallback UI.

## Error Boundary Component

```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false, error: null };
  
  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }
  
  componentDidCatch(error, info) {
    console.error('Error caught:', error, info);
  }
  
  render() {
    if (this.state.hasError) {
      return this.props.fallback || <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}
```

## Testing Error Boundary

```jsx
// Component that throws
function BrokenComponent() {
  throw new Error('Intentional error');
}

test('renders fallback when child throws', () => {
  // Silence console.error for this test
  const spy = jest.spyOn(console, 'error').mockImplementation();
  
  render(
    <ErrorBoundary fallback={<p>Error occurred</p>}>
      <BrokenComponent />
    </ErrorBoundary>
  );
  
  expect(screen.getByText('Error occurred')).toBeInTheDocument();
  
  spy.mockRestore();
});

test('renders children when no error', () => {
  render(
    <ErrorBoundary>
      <p>Working component</p>
    </ErrorBoundary>
  );
  
  expect(screen.getByText('Working component')).toBeInTheDocument();
});
```

## Testing Error Recovery

```jsx
function ErrorBoundaryWithRetry({ children }) {
  const [hasError, setHasError] = useState(false);
  
  // Reset error state
  const retry = () => setHasError(false);
  
  if (hasError) {
    return (
      <div>
        <p>Something went wrong</p>
        <button onClick={retry}>Try again</button>
      </div>
    );
  }
  
  return children;
}

test('can recover from error', async () => {
  const user = userEvent.setup();
  const spy = jest.spyOn(console, 'error').mockImplementation();
  
  let shouldThrow = true;
  function MaybeThrows() {
    if (shouldThrow) throw new Error('Error!');
    return <p>Success!</p>;
  }
  
  const { rerender } = render(
    <ErrorBoundaryWithRetry>
      <MaybeThrows />
    </ErrorBoundaryWithRetry>
  );
  
  // Shows error state
  expect(screen.getByText('Something went wrong')).toBeInTheDocument();
  
  // Fix the error condition
  shouldThrow = false;
  
  // Click retry
  await user.click(screen.getByRole('button', { name: 'Try again' }));
  
  // Now shows success
  expect(screen.getByText('Success!')).toBeInTheDocument();
  
  spy.mockRestore();
});
```

## Testing componentDidCatch

```jsx
test('logs error to service', () => {
  const logError = jest.fn();
  const spy = jest.spyOn(console, 'error').mockImplementation();
  
  render(
    <ErrorBoundary onError={logError}>
      <BrokenComponent />
    </ErrorBoundary>
  );
  
  expect(logError).toHaveBeenCalledWith(
    expect.any(Error),
    expect.objectContaining({
      componentStack: expect.any(String)
    })
  );
  
  spy.mockRestore();
});
```

---

> 📘 *This lesson is part of the [React Testing Strategies](https://stanza.dev/courses/react-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*