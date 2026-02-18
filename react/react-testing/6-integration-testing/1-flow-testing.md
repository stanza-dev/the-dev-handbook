---
source_course: "react-testing"
source_lesson: "react-testing-user-flow-testing"
---

# Testing User Flows

Integration tests verify that multiple components work together correctly.

## Complete User Journey

```jsx
test('user can complete checkout flow', async () => {
  const user = userEvent.setup();
  
  // Setup with all necessary providers
  render(
    <Providers>
      <App />
    </Providers>
  );
  
  // Step 1: Add item to cart
  await user.click(screen.getByRole('button', { name: 'Add to Cart' }));
  expect(screen.getByText('1 item in cart')).toBeInTheDocument();
  
  // Step 2: Go to cart
  await user.click(screen.getByRole('link', { name: 'Cart' }));
  expect(screen.getByRole('heading', { name: 'Shopping Cart' })).toBeInTheDocument();
  
  // Step 3: Proceed to checkout
  await user.click(screen.getByRole('button', { name: 'Checkout' }));
  
  // Step 4: Fill shipping info
  await user.type(screen.getByLabelText('Address'), '123 Main St');
  await user.type(screen.getByLabelText('City'), 'New York');
  await user.selectOptions(screen.getByLabelText('State'), 'NY');
  await user.type(screen.getByLabelText('ZIP'), '10001');
  
  await user.click(screen.getByRole('button', { name: 'Continue' }));
  
  // Step 5: Review and confirm
  expect(screen.getByText('Order Summary')).toBeInTheDocument();
  await user.click(screen.getByRole('button', { name: 'Place Order' }));
  
  // Step 6: Confirmation
  await screen.findByText('Thank you for your order!');
});
```

## Testing Navigation

```jsx
import { MemoryRouter } from 'react-router-dom';

test('navigates between pages', async () => {
  const user = userEvent.setup();
  
  render(
    <MemoryRouter initialEntries={['/']}>
      <App />
    </MemoryRouter>
  );
  
  // Start on home
  expect(screen.getByRole('heading', { name: 'Home' })).toBeInTheDocument();
  
  // Navigate to About
  await user.click(screen.getByRole('link', { name: 'About' }));
  expect(screen.getByRole('heading', { name: 'About Us' })).toBeInTheDocument();
  
  // Navigate to Contact
  await user.click(screen.getByRole('link', { name: 'Contact' }));
  expect(screen.getByRole('heading', { name: 'Contact' })).toBeInTheDocument();
});
```

## Testing with URL Parameters

```jsx
test('loads product from URL', async () => {
  render(
    <MemoryRouter initialEntries={['/products/123']}>
      <Routes>
        <Route path="/products/:id" element={<ProductPage />} />
      </Routes>
    </MemoryRouter>
  );
  
  // Waits for product to load
  await screen.findByRole('heading', { name: 'Product Name' });
});
```

## Testing Protected Routes

```jsx
test('redirects unauthenticated user to login', () => {
  render(
    <MemoryRouter initialEntries={['/dashboard']}>
      <AuthProvider user={null}>
        <App />
      </AuthProvider>
    </MemoryRouter>
  );
  
  // Should show login, not dashboard
  expect(screen.getByRole('heading', { name: 'Login' })).toBeInTheDocument();
  expect(screen.queryByRole('heading', { name: 'Dashboard' })).not.toBeInTheDocument();
});

test('shows dashboard for authenticated user', () => {
  render(
    <MemoryRouter initialEntries={['/dashboard']}>
      <AuthProvider user={{ name: 'John' }}>
        <App />
      </AuthProvider>
    </MemoryRouter>
  );
  
  expect(screen.getByRole('heading', { name: 'Dashboard' })).toBeInTheDocument();
});
```

---

> 📘 *This lesson is part of the [React Testing Strategies](https://stanza.dev/courses/react-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*