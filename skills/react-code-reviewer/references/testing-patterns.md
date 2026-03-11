# Testing Patterns Reference

## Table of Contents
1. [Component Testing Strategy](#component-testing-strategy)
2. [Testing Library Patterns](#testing-library-patterns)
3. [Hooks Testing](#hooks-testing)
4. [Async Testing](#async-testing)
5. [Common Testing Anti-Patterns](#common-testing-anti-patterns)

---

## Component Testing Strategy

### What to Test
- User interactions (clicks, typing, form submission)
- Conditional rendering based on props/state
- Side effects (API calls, localStorage)
- Error states and edge cases
- Accessibility (roles, labels)

### What NOT to Test
- Implementation details (state values, internal methods)
- Third-party library internals
- Styling (unless critical to functionality)
- Private methods or component internals

### Test Structure
```tsx
describe('ComponentName', () => {
  // Setup
  const defaultProps = { /* ... */ };
  const renderComponent = (props = {}) => 
    render(<ComponentName {...defaultProps} {...props} />);

  // Group by behavior
  describe('when user is logged in', () => {
    it('displays user name', () => { /* ... */ });
    it('shows logout button', () => { /* ... */ });
  });

  describe('when user clicks submit', () => {
    it('calls onSubmit with form data', () => { /* ... */ });
    it('shows loading state', () => { /* ... */ });
  });
});
```

---

## Testing Library Patterns

### Query Priority
```tsx
// 1. Accessible queries (preferred)
getByRole('button', { name: /submit/i })
getByLabelText('Email')
getByPlaceholderText('Search...')
getByText('Welcome back')

// 2. Semantic queries
getByAltText('User avatar')
getByTitle('Close')

// 3. Test IDs (last resort)
getByTestId('custom-element')
```

### User Events
```tsx
import userEvent from '@testing-library/user-event';

// ✅ Use userEvent for realistic interactions
it('submits form with user data', async () => {
  const user = userEvent.setup();
  const onSubmit = vi.fn();
  
  render(<LoginForm onSubmit={onSubmit} />);
  
  await user.type(screen.getByLabelText('Email'), 'test@example.com');
  await user.type(screen.getByLabelText('Password'), 'password123');
  await user.click(screen.getByRole('button', { name: /sign in/i }));
  
  expect(onSubmit).toHaveBeenCalledWith({
    email: 'test@example.com',
    password: 'password123',
  });
});
```

### Assertions
```tsx
// ✅ Prefer explicit assertions
expect(screen.getByRole('alert')).toHaveTextContent('Error occurred');
expect(screen.getByRole('button')).toBeDisabled();
expect(screen.getByRole('textbox')).toHaveValue('Hello');

// ✅ Check for absence
expect(screen.queryByText('Loading')).not.toBeInTheDocument();

// ✅ Wait for elements to appear/disappear
await waitFor(() => {
  expect(screen.queryByText('Loading')).not.toBeInTheDocument();
});
await waitForElementToBeRemoved(() => screen.queryByText('Loading'));
```

---

## Hooks Testing

### Custom Hook Testing
```tsx
import { renderHook, act } from '@testing-library/react';

describe('useCounter', () => {
  it('increments counter', () => {
    const { result } = renderHook(() => useCounter(0));
    
    expect(result.current.count).toBe(0);
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });
});
```

### Hooks with Context
```tsx
describe('useAuth', () => {
  const wrapper = ({ children }) => (
    <AuthProvider>{children}</AuthProvider>
  );
  
  it('returns current user', () => {
    const { result } = renderHook(() => useAuth(), { wrapper });
    expect(result.current.user).toBeDefined();
  });
});
```

---

## Async Testing

### API Calls
```tsx
import { rest } from 'msw';
import { setupServer } from 'msw/node';

const server = setupServer(
  rest.get('/api/users', (req, res, ctx) => {
    return res(ctx.json([{ id: 1, name: 'John' }]));
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

it('loads and displays users', async () => {
  render(<UserList />);
  
  expect(screen.getByText('Loading...')).toBeInTheDocument();
  
  await waitFor(() => {
    expect(screen.getByText('John')).toBeInTheDocument();
  });
});

it('handles error state', async () => {
  server.use(
    rest.get('/api/users', (req, res, ctx) => {
      return res(ctx.status(500));
    })
  );
  
  render(<UserList />);
  
  await waitFor(() => {
    expect(screen.getByRole('alert')).toHaveTextContent('Failed to load');
  });
});
```

### Timers
```tsx
it('shows notification then hides after 3 seconds', async () => {
  vi.useFakeTimers();
  
  render(<Toast message="Saved!" duration={3000} />);
  
  expect(screen.getByText('Saved!')).toBeInTheDocument();
  
  act(() => {
    vi.advanceTimersByTime(3000);
  });
  
  expect(screen.queryByText('Saved!')).not.toBeInTheDocument();
  
  vi.useRealTimers();
});
```

---

## Common Testing Anti-Patterns

### Anti-Pattern: Testing Implementation Details
```tsx
// ❌ Testing internal state
it('sets isOpen to true', () => {
  const { result } = renderHook(() => useState(false));
  // Don't test state values directly
});

// ✅ Test observable behavior
it('shows dropdown content when clicked', async () => {
  render(<Dropdown />);
  await userEvent.click(screen.getByRole('button'));
  expect(screen.getByRole('listbox')).toBeVisible();
});
```

### Anti-Pattern: Snapshot Overuse
```tsx
// ❌ Large component snapshots
it('renders correctly', () => {
  const { container } = render(<ComplexPage />);
  expect(container).toMatchSnapshot();
});

// ✅ Targeted assertions
it('renders user profile section', () => {
  render(<ProfilePage user={mockUser} />);
  expect(screen.getByRole('heading')).toHaveTextContent(mockUser.name);
  expect(screen.getByRole('img')).toHaveAttribute('src', mockUser.avatar);
});
```

### Anti-Pattern: Container Query Assertions
```tsx
// ❌ Querying by class or structure
expect(container.querySelector('.btn-primary')).toBeInTheDocument();

// ✅ Query by role/text
expect(screen.getByRole('button', { name: /submit/i })).toBeInTheDocument();
```

### Anti-Pattern: Not Cleaning Up
```tsx
// ❌ Pollution between tests
let sharedState;

beforeEach(() => {
  sharedState = { count: 0 };
});

// ✅ Isolated tests
it('test one', () => {
  const state = { count: 0 };
  // Use local state
});
```

### Anti-Pattern: Arbitrary Timeouts
```tsx
// ❌ Using arbitrary delays
await new Promise(r => setTimeout(r, 1000));

// ✅ Wait for specific conditions
await waitFor(() => {
  expect(screen.getByText('Loaded')).toBeInTheDocument();
});
```
