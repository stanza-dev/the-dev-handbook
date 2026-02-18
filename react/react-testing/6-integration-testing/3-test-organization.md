---
source_course: "react-testing"
source_lesson: "react-testing-test-organization"
---

# Test Organization & Maintenance

Well-organized tests are easier to maintain and understand.

## File Organization

```
src/
├── components/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx      # Unit tests
│   │   └── Button.stories.tsx   # Storybook
│   └── Form/
│       ├── Form.tsx
│       └── Form.test.tsx
├── hooks/
│   ├── useAuth.ts
│   └── useAuth.test.ts
├── pages/
│   └── Dashboard/
│       ├── Dashboard.tsx
│       └── Dashboard.test.tsx
└── __tests__/                    # Integration tests
    ├── checkout-flow.test.tsx
    └── user-registration.test.tsx
```

## describe Blocks

```jsx
describe('LoginForm', () => {
  describe('validation', () => {
    test('shows error for empty email', async () => {
      // ...
    });
    
    test('shows error for invalid email format', async () => {
      // ...
    });
  });
  
  describe('submission', () => {
    test('calls onSubmit with credentials', async () => {
      // ...
    });
    
    test('shows error on failed login', async () => {
      // ...
    });
  });
  
  describe('accessibility', () => {
    test('inputs have labels', () => {
      // ...
    });
    
    test('errors are announced to screen readers', async () => {
      // ...
    });
  });
});
```

## Shared Setup

```jsx
describe('UserProfile', () => {
  let user;
  
  beforeEach(() => {
    user = userEvent.setup();
  });
  
  const renderProfile = (props = {}) => {
    return render(
      <UserProfile
        user={{ name: 'John', email: 'john@test.com' }}
        {...props}
      />
    );
  };
  
  test('displays user name', () => {
    renderProfile();
    expect(screen.getByText('John')).toBeInTheDocument();
  });
  
  test('displays custom user', () => {
    renderProfile({ user: { name: 'Jane', email: 'jane@test.com' } });
    expect(screen.getByText('Jane')).toBeInTheDocument();
  });
});
```

## Test Naming Conventions

```jsx
// ✅ Good: Describes behavior from user perspective
test('shows success message after form submission', async () => {});
test('disables submit button while loading', async () => {});
test('navigates to dashboard after login', async () => {});

// ❌ Bad: Implementation-focused names
test('calls setLoading', async () => {});
test('updates state correctly', async () => {});
test('renders component', () => {});
```

## Custom Test Utilities

```jsx
// test-utils/index.js
export * from '@testing-library/react';
export { default as userEvent } from '@testing-library/user-event';

// Custom render with providers
export function renderWithProviders(ui, options = {}) {
  const Wrapper = ({ children }) => (
    <QueryClientProvider client={createTestQueryClient()}>
      <AuthProvider>
        <ThemeProvider>
          <MemoryRouter>
            {children}
          </MemoryRouter>
        </ThemeProvider>
      </AuthProvider>
    </QueryClientProvider>
  );
  
  return render(ui, { wrapper: Wrapper, ...options });
}

// Test data factories
export function createUser(overrides = {}) {
  return {
    id: 'user-1',
    name: 'Test User',
    email: 'test@example.com',
    ...overrides
  };
}
```

---

> 📘 *This lesson is part of the [React Testing Strategies](https://stanza.dev/courses/react-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*