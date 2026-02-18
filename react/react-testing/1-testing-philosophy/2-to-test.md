---
source_course: "react-testing"
source_lesson: "react-testing-what-to-test"
---

# What to Test (and What Not To)

Knowing what to test is as important as knowing how to test.

## Test User Behavior, Not Implementation

### ❌ Don't Test Implementation Details

```jsx
// Bad: Testing internal state
test('sets isLoading to true', () => {
  const { result } = renderHook(() => useData());
  act(() => result.current.fetchData());
  expect(result.current.isLoading).toBe(true);
});
```

### ✅ Do Test User-Visible Behavior

```jsx
// Good: Testing what user sees
test('shows loading spinner while fetching', () => {
  render(<DataList />);
  fireEvent.click(screen.getByText('Load Data'));
  expect(screen.getByRole('status')).toBeInTheDocument();
});
```

## What You SHOULD Test

### 1. User Interactions

```jsx
test('clicking add button adds item to list', () => {
  render(<TodoApp />);
  fireEvent.type(screen.getByPlaceholderText('New todo'), 'Buy milk');
  fireEvent.click(screen.getByText('Add'));
  expect(screen.getByText('Buy milk')).toBeInTheDocument();
});
```

### 2. Conditional Rendering

```jsx
test('shows login form when not authenticated', () => {
  render(<App isAuthenticated={false} />);
  expect(screen.getByRole('form', { name: /login/i })).toBeInTheDocument();
});

test('shows dashboard when authenticated', () => {
  render(<App isAuthenticated={true} />);
  expect(screen.getByRole('heading', { name: /dashboard/i })).toBeInTheDocument();
});
```

### 3. Error States

```jsx
test('displays error message on failed submission', async () => {
  server.use(rest.post('/api/submit', (req, res, ctx) => 
    res(ctx.status(500))
  ));
  
  render(<ContactForm />);
  fireEvent.click(screen.getByText('Submit'));
  
  await screen.findByText(/something went wrong/i);
});
```

### 4. Accessibility

```jsx
test('form inputs have accessible labels', () => {
  render(<SignupForm />);
  expect(screen.getByLabelText('Email')).toBeInTheDocument();
  expect(screen.getByLabelText('Password')).toBeInTheDocument();
});
```

## What You Should NOT Test

### 1. Third-Party Libraries

```jsx
// Don't test that React Router works
test('Link renders correctly', () => {
  render(<Link to="/about">About</Link>);
  // This tests react-router, not your code
});
```

### 2. Implementation Details

```jsx
// Don't test internal state names
test('useState is called', () => {
  // This couples tests to implementation
});
```

### 3. Styling (Usually)

```jsx
// Don't test CSS classes
test('has blue background', () => {
  expect(element).toHaveClass('bg-blue-500');
  // Brittle - class names change
});
```

## The Golden Rule

> Test your code the way your users use it.

If you can describe a test starting with "When the user...", it's probably a good test.

---

> 📘 *This lesson is part of the [React Testing Strategies](https://stanza.dev/courses/react-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*