---
source_course: "react-components-patterns"
source_lesson: "react-components-patterns-error-strategies"
---

# Error Handling Strategies

Build resilient applications with comprehensive error handling.

## Granular Error Boundaries

Place boundaries strategically:

```tsx
function Dashboard() {
  return (
    <div className="dashboard">
      {/* Critical - show full error */}
      <ErrorBoundary fallback={<CriticalError />}>
        <Navigation />
      </ErrorBoundary>
      
      {/* Non-critical - show placeholder */}
      <div className="widgets">
        <ErrorBoundary fallback={<WidgetPlaceholder />}>
          <RevenueWidget />
        </ErrorBoundary>
        
        <ErrorBoundary fallback={<WidgetPlaceholder />}>
          <UsersWidget />
        </ErrorBoundary>
        
        <ErrorBoundary fallback={<WidgetPlaceholder />}>
          <ActivityWidget />
        </ErrorBoundary>
      </div>
    </div>
  );
}
```

## Reset on Navigation

```tsx
import { useLocation } from 'react-router-dom';

function App() {
  const location = useLocation();
  
  return (
    <ErrorBoundary
      key={location.pathname} // Reset on route change
      fallback={<ErrorPage />}
    >
      <Routes>
        {/* ... */}
      </Routes>
    </ErrorBoundary>
  );
}
```

## Retry Logic

```tsx
type RetryBoundaryProps = {
  children: ReactNode;
  maxRetries?: number;
};

function RetryBoundary({ children, maxRetries = 3 }: RetryBoundaryProps) {
  const [retryCount, setRetryCount] = useState(0);
  const [error, setError] = useState<Error | null>(null);
  
  if (error) {
    if (retryCount < maxRetries) {
      return (
        <div className="error-retry">
          <p>Something went wrong. Retrying... ({retryCount + 1}/{maxRetries})</p>
          <button onClick={() => {
            setRetryCount(c => c + 1);
            setError(null);
          }}>
            Retry Now
          </button>
        </div>
      );
    }
    
    return (
      <div className="error-final">
        <p>Failed after {maxRetries} attempts.</p>
        <button onClick={() => {
          setRetryCount(0);
          setError(null);
        }}>
          Start Over
        </button>
      </div>
    );
  }
  
  return (
    <ErrorBoundary
      fallback={null}
      onError={(err) => setError(err)}
    >
      {children}
    </ErrorBoundary>
  );
}
```

## Async Error Handling

```tsx
function useAsyncError() {
  const [, setError] = useState();
  
  return useCallback((error: Error) => {
    setError(() => {
      throw error;
    });
  }, []);
}

function DataFetcher() {
  const throwError = useAsyncError();
  
  useEffect(() => {
    fetchData()
      .then(setData)
      .catch(throwError); // Triggers error boundary
  }, [throwError]);
  
  return <div>{/* ... */}</div>;
}
```

## Error Reporting

```tsx
function reportError(error: Error, errorInfo: ErrorInfo) {
  // Send to error tracking service
  fetch('/api/errors', {
    method: 'POST',
    body: JSON.stringify({
      message: error.message,
      stack: error.stack,
      componentStack: errorInfo.componentStack,
      url: window.location.href,
      userAgent: navigator.userAgent,
      timestamp: new Date().toISOString()
    })
  });
}

class ReportingErrorBoundary extends Component<Props, State> {
  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    reportError(error, errorInfo);
  }
  
  // ...
}
```

## User-Friendly Error Messages

```tsx
function ErrorFallback({ error, resetErrorBoundary }: FallbackProps) {
  const isDev = process.env.NODE_ENV === 'development';
  
  return (
    <div className="error-fallback" role="alert">
      <h2>Oops! Something went wrong</h2>
      <p>
        We're sorry for the inconvenience. Please try refreshing the page
        or contact support if the problem persists.
      </p>
      
      {isDev && (
        <details>
          <summary>Error Details (Development Only)</summary>
          <pre>{error.message}</pre>
          <pre>{error.stack}</pre>
        </details>
      )}
      
      <div className="error-actions">
        <button onClick={resetErrorBoundary}>Try Again</button>
        <button onClick={() => window.location.reload()}>Refresh Page</button>
        <a href="/support">Contact Support</a>
      </div>
    </div>
  );
}
```

## Resources

- [react-error-boundary](https://github.com/bvaughn/react-error-boundary) — Popular error boundary library with hooks support

---

> 📘 *This lesson is part of the [React Components & Patterns](https://stanza.dev/courses/react-components-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*