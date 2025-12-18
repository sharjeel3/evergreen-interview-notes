# Test-Driven Development (TDD) and Test Types

**Level:** 🔥 Advanced  
**Tags:** `tdd`, `testing`, `test-types`, `jest`, `cypress`, `playwright`, `interview`

---

## Why this matters

Test-Driven Development is a disciplined approach that fundamentally changes how you write code. Instead of writing tests after implementation, TDD requires you to write tests first, driving your design and ensuring every line of code has a purpose.

**Why interviewers care:**
- **Engineering discipline:** TDD demonstrates you write deliberate, thoughtful code rather than "cowboy coding"
- **Design skills:** Writing tests first forces you to think about APIs, interfaces, and contracts before implementation
- **Quality mindset:** Shows you care about correctness, maintainability, and preventing bugs
- **Refactoring confidence:** Tests provide safety net for code improvements and architecture changes
- **Senior-level skill:** TDD is expected at senior+ levels where code quality and long-term maintainability matter
- **Real-world practice:** Many top companies (Google, Microsoft, Spotify) emphasize testing culture
- **Communication:** Tests document expected behavior better than comments ever could

**Real-world benefits:**
- **Fewer bugs:** Catch issues before they reach production (10-100x cheaper to fix)
- **Better design:** Tests force you to write decoupled, testable code
- **Faster debugging:** When tests fail, you know exactly what broke
- **Refactoring safety:** Change code confidently without breaking existing functionality
- **Living documentation:** Tests show how code should be used
- **Team confidence:** New team members can modify code without fear
- **Faster onboarding:** Tests teach new developers how the system works
- **Reduced QA cycles:** Automated tests catch regressions immediately

**Common TDD challenges:**
- Initial slowdown (writing tests takes time upfront)
- Learning curve for testing tools and patterns
- Resistance from teams unfamiliar with TDD
- Over-testing (testing implementation details instead of behavior)
- Maintaining test suites as codebase grows
- Testing complex UI interactions and async behavior
- Knowing what to test and what to skip

**What you must know:**
- **The TDD cycle:** Red-Green-Refactor workflow
- **Test types:** When to use unit, integration, E2E, etc.
- **Testing pyramid:** Distribution of test types for optimal coverage
- **Testing philosophy:** Test behavior, not implementation
- **React testing tools:** Jest, React Testing Library, Cypress, Playwright
- **Best practices:** AAA pattern, isolation, deterministic tests
- **Coverage vs. confidence:** 100% coverage ≠ bug-free code

**Interview red flags:** Never having practiced TDD, testing implementation details, writing tests after code is "done", or believing tests slow down development.

---

## Core ideas

### What is Test-Driven Development (TDD)?

TDD is a software development methodology where you write tests **before** writing the actual implementation code. It's not just about testing—it's a design and development discipline that shapes how you build software.

**Key principles:**
- **Tests first:** Write failing tests before any production code
- **Minimal code:** Write only enough code to make tests pass
- **Continuous refactoring:** Improve code structure while keeping tests green
- **Fast feedback:** Run tests frequently to catch issues immediately
- **Design driver:** Tests guide your API design and architecture

**TDD is NOT:**
- Writing tests after code is done (that's just regular testing)
- Testing every possible edge case upfront
- A replacement for QA or integration testing
- A guarantee of bug-free code
- Required for every single line of code

**When to use TDD:**
- ✅ Business logic and algorithms
- ✅ Complex React components with multiple states
- ✅ Custom hooks with non-trivial logic
- ✅ Utility functions and helpers
- ✅ State management (reducers, stores)
- ✅ Form validation logic
- ✅ API integration layers
- ✅ Critical user flows

**When TDD might not fit:**
- ❌ Quick prototypes or proof-of-concepts
- ❌ Simple presentational components (just props in, JSX out)
- ❌ Exploratory coding where requirements are unclear
- ❌ UI layout and styling (use visual regression instead)
- ❌ One-off scripts or migrations

### The TDD Cycle: Red-Green-Refactor

The heart of TDD is a simple three-step cycle repeated hundreds of times during development:

#### 🔴 Red: Write a Failing Test

1. **Write a test** for a small piece of functionality that doesn't exist yet
2. **Run the test** and watch it fail (red)
3. **Verify the failure** is for the right reason

```javascript
// Example: Testing a custom useToggle hook (doesn't exist yet)
import { renderHook, act } from '@testing-library/react';
import { useToggle } from './useToggle';

test('should toggle boolean value', () => {
  const { result } = renderHook(() => useToggle(false));
  
  expect(result.current.value).toBe(false);
  
  act(() => {
    result.current.toggle();
  });
  
  expect(result.current.value).toBe(true);
});

// This test WILL FAIL because useToggle doesn't exist yet
```

**Why start with failure?**
- Confirms the test can actually detect problems
- Prevents false positives (tests that pass but don't test anything)
- Forces you to think about what you're building before building it

#### 🟢 Green: Make the Test Pass

1. **Write minimal code** to make the test pass
2. **Don't worry about perfection**—just make it work
3. **Run the test** and verify it passes (green)

```javascript
// useToggle.js - Minimal implementation
import { useState } from 'react';

export function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);
  
  const toggle = () => {
    setValue(!value);
  };
  
  return { value, toggle };
}

// Test now PASSES ✅
```

**Why minimal code?**
- Prevents over-engineering
- Keeps you focused on current requirement
- Reveals when you're adding untested code

#### 🔵 Refactor: Improve the Code

1. **Clean up the code** while keeping tests green
2. **Improve design** without changing behavior
3. **Run tests** after each change to ensure nothing broke

```javascript
// useToggle.js - Refactored with additional features
import { useState, useCallback } from 'react';

export function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);
  
  const toggle = useCallback(() => {
    setValue(v => !v);
  }, []);
  
  const setTrue = useCallback(() => setValue(true), []);
  const setFalse = useCallback(() => setValue(false), []);
  
  return { value, toggle, setTrue, setFalse };
}

// Tests still PASS ✅ after refactoring
```

**What to refactor:**
- Extract duplicated code
- Improve variable names
- Simplify complex logic
- Add performance optimizations (useMemo, useCallback)
- Improve code organization

**🔄 Repeat the cycle:**
1. Red: Write next test for new behavior
2. Green: Implement minimal code
3. Refactor: Clean up
4. Red: Write next test...

### Benefits of the Red-Green-Refactor Cycle

| Benefit | How TDD Helps |
|---------|---------------|
| **Confidence** | Tests prove code works as expected |
| **Focus** | One small feature at a time |
| **Design** | Tests reveal tight coupling and bad design |
| **Documentation** | Tests show how to use the code |
| **Debugging** | Small iterations make bugs obvious |
| **Regression prevention** | Old tests catch new bugs |
| **Refactoring safety** | Change code without breaking features |

### TDD in React: Practical Workflow

```javascript
// 1. RED: Write failing test for UserProfile component
test('renders user name and email', () => {
  const user = { name: 'Alice', email: 'alice@example.com' };
  render(<UserProfile user={user} />);
  
  expect(screen.getByText('Alice')).toBeInTheDocument();
  expect(screen.getByText('alice@example.com')).toBeInTheDocument();
});
// ❌ FAILS: UserProfile doesn't exist

// 2. GREEN: Minimal implementation
function UserProfile({ user }) {
  return (
    <div>
      <div>{user.name}</div>
      <div>{user.email}</div>
    </div>
  );
}
// ✅ PASSES

// 3. REFACTOR: Improve styling and structure
function UserProfile({ user }) {
  return (
    <div className="user-profile">
      <h2 className="user-name">{user.name}</h2>
      <p className="user-email">{user.email}</p>
    </div>
  );
}
// ✅ STILL PASSES after refactoring

// 4. RED: Add test for loading state
test('shows loading spinner when user is loading', () => {
  render(<UserProfile user={null} isLoading={true} />);
  expect(screen.getByRole('status')).toBeInTheDocument();
});
// ❌ FAILS: No loading state handling

// Continue the cycle...
```

---

## Test Types Overview

### 1. Unit Tests

**Definition:** Tests that verify individual functions, methods, or utilities in isolation, independent of React components or external dependencies.

**Purpose:** Ensure small, focused pieces of logic work correctly on their own.

**When to use:**
- ✅ Pure functions (calculations, transformations, validations)
- ✅ Utility functions (formatters, parsers, helpers)
- ✅ Custom algorithms and business logic
- ✅ Data transformations
- ✅ Validators and sanitizers
- ✅ Constants and configuration logic

**Tools:**
- **Jest:** Test runner, assertions, mocking
- **Vitest:** Faster Jest alternative, ESM-native
- **@testing-library/jest-dom:** Custom matchers for DOM

**Example 1: Testing a pure utility function**

```javascript
// utils/currency.js
export function formatCurrency(amount, currency = 'USD') {
  if (typeof amount !== 'number') {
    throw new Error('Amount must be a number');
  }
  
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency,
  }).format(amount);
}

// utils/currency.test.js
import { formatCurrency } from './currency';

describe('formatCurrency', () => {
  test('formats USD currency correctly', () => {
    expect(formatCurrency(1234.56)).toBe('$1,234.56');
  });
  
  test('formats EUR currency correctly', () => {
    expect(formatCurrency(1234.56, 'EUR')).toBe('€1,234.56');
  });
  
  test('handles zero amount', () => {
    expect(formatCurrency(0)).toBe('$0.00');
  });
  
  test('handles negative amounts', () => {
    expect(formatCurrency(-50.99)).toBe('-$50.99');
  });
  
  test('throws error for non-number input', () => {
    expect(() => formatCurrency('invalid')).toThrow('Amount must be a number');
  });
});
```

**Example 2: Testing validation logic**

```javascript
// utils/validation.js
export function validateEmail(email) {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}

export function validatePassword(password) {
  return password.length >= 8 && /[A-Z]/.test(password) && /[0-9]/.test(password);
}

// utils/validation.test.js
import { validateEmail, validatePassword } from './validation';

describe('validateEmail', () => {
  test('accepts valid email addresses', () => {
    expect(validateEmail('user@example.com')).toBe(true);
    expect(validateEmail('test.user+tag@domain.co.uk')).toBe(true);
  });
  
  test('rejects invalid email addresses', () => {
    expect(validateEmail('invalid')).toBe(false);
    expect(validateEmail('@example.com')).toBe(false);
    expect(validateEmail('user@')).toBe(false);
    expect(validateEmail('')).toBe(false);
  });
});

describe('validatePassword', () => {
  test('accepts valid passwords', () => {
    expect(validatePassword('Password1')).toBe(true);
    expect(validatePassword('MySecure123')).toBe(true);
  });
  
  test('rejects passwords without uppercase', () => {
    expect(validatePassword('password1')).toBe(false);
  });
  
  test('rejects passwords without numbers', () => {
    expect(validatePassword('Password')).toBe(false);
  });
  
  test('rejects short passwords', () => {
    expect(validatePassword('Pass1')).toBe(false);
  });
});
```

**Example 3: Testing data transformations**

```javascript
// utils/arrays.js
export function groupBy(array, key) {
  return array.reduce((result, item) => {
    const group = item[key];
    if (!result[group]) {
      result[group] = [];
    }
    result[group].push(item);
    return result;
  }, {});
}

// utils/arrays.test.js
import { groupBy } from './arrays';

describe('groupBy', () => {
  test('groups objects by key', () => {
    const users = [
      { name: 'Alice', role: 'admin' },
      { name: 'Bob', role: 'user' },
      { name: 'Charlie', role: 'admin' },
    ];
    
    const grouped = groupBy(users, 'role');
    
    expect(grouped).toEqual({
      admin: [
        { name: 'Alice', role: 'admin' },
        { name: 'Charlie', role: 'admin' },
      ],
      user: [
        { name: 'Bob', role: 'user' },
      ],
    });
  });
  
  test('handles empty array', () => {
    expect(groupBy([], 'key')).toEqual({});
  });
  
  test('handles single group', () => {
    const items = [{ id: 1, type: 'A' }, { id: 2, type: 'A' }];
    const grouped = groupBy(items, 'type');
    
    expect(grouped.A).toHaveLength(2);
  });
});
```

**Best Practices:**
- ✅ Test one thing per test
- ✅ Use descriptive test names
- ✅ Follow AAA pattern (Arrange, Act, Assert)
- ✅ Test edge cases and error conditions
- ✅ Keep tests fast and isolated
- ❌ Don't test third-party libraries
- ❌ Don't test implementation details

---

### 2. Component Tests

**Definition:** Tests that verify React component behavior by rendering them and simulating user interactions, focusing on what users see and do.

**Purpose:** Ensure components render correctly and respond to user interactions as expected.

**When to use:**
- ✅ Any user-facing React component
- ✅ Components with user interactions (clicks, inputs)
- ✅ Components with conditional rendering
- ✅ Components with multiple states
- ✅ Form components
- ✅ Components that integrate with hooks

**Tools:**
- **React Testing Library (RTL):** Recommended by React team, user-centric testing
- **@testing-library/react:** Core RTL package
- **@testing-library/user-event:** Realistic user interaction simulation
- **@testing-library/jest-dom:** DOM matchers
- **Jest/Vitest:** Test runner

**Philosophy:** Test components like users use them—through the UI, not internal state.

**Example 1: Testing a Button component**

```javascript
// Button.jsx
export function Button({ onClick, disabled, children }) {
  return (
    <button onClick={onClick} disabled={disabled}>
      {children}
    </button>
  );
}

// Button.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Button } from './Button';

describe('Button', () => {
  test('renders button with text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button', { name: 'Click me' })).toBeInTheDocument();
  });
  
  test('calls onClick when clicked', async () => {
    const user = userEvent.setup();
    const handleClick = jest.fn();
    
    render(<Button onClick={handleClick}>Click me</Button>);
    
    await user.click(screen.getByRole('button'));
    
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
  
  test('does not call onClick when disabled', async () => {
    const user = userEvent.setup();
    const handleClick = jest.fn();
    
    render(<Button onClick={handleClick} disabled>Click me</Button>);
    
    await user.click(screen.getByRole('button'));
    
    expect(handleClick).not.toHaveBeenCalled();
  });
  
  test('is disabled when disabled prop is true', () => {
    render(<Button disabled>Click me</Button>);
    expect(screen.getByRole('button')).toBeDisabled();
  });
});
```

**Example 2: Testing a Counter component with state**

```javascript
// Counter.jsx
import { useState } from 'react';

export function Counter({ initialCount = 0 }) {
  const [count, setCount] = useState(initialCount);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={() => setCount(count - 1)}>Decrement</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}

// Counter.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Counter } from './Counter';

describe('Counter', () => {
  test('renders with initial count', () => {
    render(<Counter initialCount={5} />);
    expect(screen.getByText('Count: 5')).toBeInTheDocument();
  });
  
  test('increments count when increment button is clicked', async () => {
    const user = userEvent.setup();
    render(<Counter />);
    
    await user.click(screen.getByRole('button', { name: 'Increment' }));
    
    expect(screen.getByText('Count: 1')).toBeInTheDocument();
  });
  
  test('decrements count when decrement button is clicked', async () => {
    const user = userEvent.setup();
    render(<Counter initialCount={5} />);
    
    await user.click(screen.getByRole('button', { name: 'Decrement' }));
    
    expect(screen.getByText('Count: 4')).toBeInTheDocument();
  });
  
  test('resets count to zero', async () => {
    const user = userEvent.setup();
    render(<Counter initialCount={10} />);
    
    await user.click(screen.getByRole('button', { name: 'Reset' }));
    
    expect(screen.getByText('Count: 0')).toBeInTheDocument();
  });
  
  test('handles multiple interactions', async () => {
    const user = userEvent.setup();
    render(<Counter />);
    
    await user.click(screen.getByRole('button', { name: 'Increment' }));
    await user.click(screen.getByRole('button', { name: 'Increment' }));
    await user.click(screen.getByRole('button', { name: 'Decrement' }));
    
    expect(screen.getByText('Count: 1')).toBeInTheDocument();
  });
});
```

**Example 3: Testing a LoginForm component**

```javascript
// LoginForm.jsx
import { useState } from 'react';

export function LoginForm({ onSubmit }) {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault();
    
    if (!email || !password) {
      setError('Email and password are required');
      return;
    }
    
    setError('');
    onSubmit({ email, password });
  };
  
  return (
    <form onSubmit={handleSubmit} aria-label="Login form">
      {error && <div role="alert">{error}</div>}
      
      <label htmlFor="email">Email</label>
      <input
        id="email"
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />
      
      <label htmlFor="password">Password</label>
      <input
        id="password"
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
      />
      
      <button type="submit">Login</button>
    </form>
  );
}

// LoginForm.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { LoginForm } from './LoginForm';

describe('LoginForm', () => {
  test('renders form fields', () => {
    render(<LoginForm onSubmit={jest.fn()} />);
    
    expect(screen.getByLabelText('Email')).toBeInTheDocument();
    expect(screen.getByLabelText('Password')).toBeInTheDocument();
    expect(screen.getByRole('button', { name: 'Login' })).toBeInTheDocument();
  });
  
  test('submits form with valid data', async () => {
    const user = userEvent.setup();
    const handleSubmit = jest.fn();
    
    render(<LoginForm onSubmit={handleSubmit} />);
    
    await user.type(screen.getByLabelText('Email'), 'user@example.com');
    await user.type(screen.getByLabelText('Password'), 'password123');
    await user.click(screen.getByRole('button', { name: 'Login' }));
    
    expect(handleSubmit).toHaveBeenCalledWith({
      email: 'user@example.com',
      password: 'password123',
    });
  });
  
  test('shows error when submitting empty form', async () => {
    const user = userEvent.setup();
    const handleSubmit = jest.fn();
    
    render(<LoginForm onSubmit={handleSubmit} />);
    
    await user.click(screen.getByRole('button', { name: 'Login' }));
    
    expect(screen.getByRole('alert')).toHaveTextContent('Email and password are required');
    expect(handleSubmit).not.toHaveBeenCalled();
  });
  
  test('updates input values as user types', async () => {
    const user = userEvent.setup();
    render(<LoginForm onSubmit={jest.fn()} />);
    
    const emailInput = screen.getByLabelText('Email');
    await user.type(emailInput, 'test@example.com');
    
    expect(emailInput).toHaveValue('test@example.com');
  });
});
```

**Example 4: Testing conditional rendering**

```javascript
// UserStatus.jsx
export function UserStatus({ user, isLoading, error }) {
  if (isLoading) {
    return <div role="status">Loading...</div>;
  }
  
  if (error) {
    return <div role="alert">Error: {error}</div>;
  }
  
  if (!user) {
    return <div>No user found</div>;
  }
  
  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}

// UserStatus.test.jsx
import { render, screen } from '@testing-library/react';
import { UserStatus } from './UserStatus';

describe('UserStatus', () => {
  test('shows loading state', () => {
    render(<UserStatus isLoading={true} />);
    expect(screen.getByRole('status')).toHaveTextContent('Loading...');
  });
  
  test('shows error state', () => {
    render(<UserStatus error="Failed to load user" />);
    expect(screen.getByRole('alert')).toHaveTextContent('Error: Failed to load user');
  });
  
  test('shows no user message', () => {
    render(<UserStatus user={null} />);
    expect(screen.getByText('No user found')).toBeInTheDocument();
  });
  
  test('shows user information', () => {
    const user = { name: 'Alice', email: 'alice@example.com' };
    render(<UserStatus user={user} />);
    
    expect(screen.getByRole('heading')).toHaveTextContent('Alice');
    expect(screen.getByText('alice@example.com')).toBeInTheDocument();
  });
});
```

**RTL Query Priority (use in this order):**

1. **getByRole** - Most accessible, preferred
2. **getByLabelText** - Forms and labels
3. **getByPlaceholderText** - If no label exists
4. **getByText** - Non-interactive elements
5. **getByDisplayValue** - Form elements with values
6. **getByAltText** - Images with alt text
7. **getByTitle** - Last resort
8. **getByTestId** - Only when nothing else works

**Query Variants:**
- `getBy*` - Element must exist (throws error if not found)
- `queryBy*` - Element may not exist (returns null)
- `findBy*` - Element will appear asynchronously (returns promise)

**Best Practices:**
- ✅ Test user-visible behavior, not implementation
- ✅ Use accessible queries (getByRole, getByLabelText)
- ✅ Use user-event for realistic interactions
- ✅ Test error states and edge cases
- ✅ Keep tests maintainable and readable
- ❌ Don't test internal component state
- ❌ Don't use test IDs unless absolutely necessary
- ❌ Don't test props directly

---

### 3. Integration Tests

**Definition:** Tests that verify multiple components, modules, or layers work together correctly. Integration tests focus on interactions between different parts of your application.

**Purpose:** Catch bugs that occur when separate pieces are combined, even if each piece works individually.

**When to use:**
- ✅ Multiple components working together
- ✅ Component + custom hook integration
- ✅ Data fetching + component rendering
- ✅ Form submission flow (UI → validation → API)
- ✅ Context providers + consuming components
- ✅ Router navigation flows
- ✅ Real API interactions (with test databases)

**Tools:**
- **React Testing Library:** Render multiple components together
- **MSW (Mock Service Worker):** Mock API calls at network level
- **Jest/Vitest:** Test runner
- **Testing containers:** Isolated test environments

**Example 1: Testing component + custom hook integration**

```javascript
// hooks/useUser.js
import { useState, useEffect } from 'react';

export function useUser(userId) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    if (!userId) return;
    
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        setUser(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, [userId]);
  
  return { user, loading, error };
}

// UserProfile.jsx
import { useUser } from './hooks/useUser';

export function UserProfile({ userId }) {
  const { user, loading, error } = useUser(userId);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return <div>User not found</div>;
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}

// UserProfile.integration.test.jsx
import { render, screen, waitFor } from '@testing-library/react';
import { http, HttpResponse } from 'msw';
import { setupServer } from 'msw/node';
import { UserProfile } from './UserProfile';

// Setup MSW server to intercept API calls
const server = setupServer(
  http.get('/api/users/:userId', ({ params }) => {
    const { userId } = params;
    
    if (userId === '1') {
      return HttpResponse.json({
        id: 1,
        name: 'Alice',
        email: 'alice@example.com',
      });
    }
    
    return new HttpResponse(null, { status: 404 });
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

describe('UserProfile Integration', () => {
  test('fetches and displays user data', async () => {
    render(<UserProfile userId="1" />);
    
    // Initially shows loading
    expect(screen.getByText('Loading...')).toBeInTheDocument();
    
    // Wait for data to load
    await waitFor(() => {
      expect(screen.getByText('Alice')).toBeInTheDocument();
    });
    
    expect(screen.getByText('alice@example.com')).toBeInTheDocument();
  });
  
  test('handles user not found', async () => {
    render(<UserProfile userId="999" />);
    
    await waitFor(() => {
      expect(screen.getByText('User not found')).toBeInTheDocument();
    });
  });
  
  test('handles API errors', async () => {
    server.use(
      http.get('/api/users/:userId', () => {
        return new HttpResponse(null, { status: 500 });
      })
    );
    
    render(<UserProfile userId="1" />);
    
    await waitFor(() => {
      expect(screen.getByText(/Error/)).toBeInTheDocument();
    });
  });
});
```

**Example 2: Testing Context + Components integration**

```javascript
// AuthContext.jsx
import { createContext, useContext, useState } from 'react';

const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  
  const login = (userData) => setUser(userData);
  const logout = () => setUser(null);
  
  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

export const useAuth = () => useContext(AuthContext);

// LoginButton.jsx
import { useAuth } from './AuthContext';

export function LoginButton() {
  const { user, login, logout } = useAuth();
  
  if (user) {
    return (
      <div>
        <span>Welcome, {user.name}</span>
        <button onClick={logout}>Logout</button>
      </div>
    );
  }
  
  return (
    <button onClick={() => login({ name: 'Alice', id: 1 })}>
      Login
    </button>
  );
}

// AuthFlow.integration.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { AuthProvider } from './AuthContext';
import { LoginButton } from './LoginButton';

describe('Auth Flow Integration', () => {
  test('login and logout flow works correctly', async () => {
    const user = userEvent.setup();
    
    render(
      <AuthProvider>
        <LoginButton />
      </AuthProvider>
    );
    
    // Initially not logged in
    expect(screen.getByRole('button', { name: 'Login' })).toBeInTheDocument();
    
    // Click login
    await user.click(screen.getByRole('button', { name: 'Login' }));
    
    // Now logged in
    expect(screen.getByText('Welcome, Alice')).toBeInTheDocument();
    expect(screen.getByRole('button', { name: 'Logout' })).toBeInTheDocument();
    
    // Click logout
    await user.click(screen.getByRole('button', { name: 'Logout' }));
    
    // Back to logged out state
    expect(screen.getByRole('button', { name: 'Login' })).toBeInTheDocument();
  });
});
```

**Example 3: Testing form submission with API**

```javascript
// CreateUserForm.jsx
import { useState } from 'react';

export function CreateUserForm({ onSuccess }) {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [status, setStatus] = useState('idle');
  const [error, setError] = useState(null);
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    setStatus('loading');
    setError(null);
    
    try {
      const response = await fetch('/api/users', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ name, email }),
      });
      
      if (!response.ok) throw new Error('Failed to create user');
      
      const user = await response.json();
      setStatus('success');
      onSuccess(user);
    } catch (err) {
      setStatus('error');
      setError(err.message);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {error && <div role="alert">{error}</div>}
      
      <input
        placeholder="Name"
        value={name}
        onChange={(e) => setName(e.target.value)}
      />
      <input
        type="email"
        placeholder="Email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />
      
      <button type="submit" disabled={status === 'loading'}>
        {status === 'loading' ? 'Creating...' : 'Create User'}
      </button>
      
      {status === 'success' && <div>User created successfully!</div>}
    </form>
  );
}

// CreateUserForm.integration.test.jsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { http, HttpResponse } from 'msw';
import { setupServer } from 'msw/node';
import { CreateUserForm } from './CreateUserForm';

const server = setupServer(
  http.post('/api/users', async ({ request }) => {
    const body = await request.json();
    return HttpResponse.json({
      id: 1,
      ...body,
    });
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

describe('CreateUserForm Integration', () => {
  test('complete user creation flow', async () => {
    const user = userEvent.setup();
    const onSuccess = jest.fn();
    
    render(<CreateUserForm onSuccess={onSuccess} />);
    
    // Fill form
    await user.type(screen.getByPlaceholderText('Name'), 'Bob');
    await user.type(screen.getByPlaceholderText('Email'), 'bob@example.com');
    
    // Submit
    await user.click(screen.getByRole('button', { name: 'Create User' }));
    
    // Shows loading state
    expect(screen.getByRole('button', { name: 'Creating...' })).toBeDisabled();
    
    // Wait for success
    await waitFor(() => {
      expect(screen.getByText('User created successfully!')).toBeInTheDocument();
    });
    
    expect(onSuccess).toHaveBeenCalledWith({
      id: 1,
      name: 'Bob',
      email: 'bob@example.com',
    });
  });
  
  test('handles API errors', async () => {
    const user = userEvent.setup();
    
    server.use(
      http.post('/api/users', () => {
        return new HttpResponse(null, { status: 500 });
      })
    );
    
    render(<CreateUserForm onSuccess={jest.fn()} />);
    
    await user.type(screen.getByPlaceholderText('Name'), 'Bob');
    await user.type(screen.getByPlaceholderText('Email'), 'bob@example.com');
    await user.click(screen.getByRole('button'));
    
    await waitFor(() => {
      expect(screen.getByRole('alert')).toHaveTextContent('Failed to create user');
    });
  });
});
```

**Best Practices:**
- ✅ Test realistic user workflows
- ✅ Mock external dependencies (APIs, databases)
- ✅ Test happy path + error cases
- ✅ Use MSW for API mocking (more realistic than jest.mock)
- ✅ Test component interactions, not isolation
- ✅ Keep tests focused on one workflow
- ❌ Don't test every permutation (that's for unit tests)
- ❌ Don't make tests too large or complex

---

### 4. End-to-End (E2E) Tests

**Definition:** Tests that simulate real user scenarios in a real browser, testing the entire application from frontend to backend.

**Purpose:** Verify critical user journeys work correctly in a production-like environment.

**When to use:**
- ✅ Critical user flows (signup, checkout, login)
- ✅ Multi-page workflows
- ✅ Cross-browser compatibility testing
- ✅ Real authentication flows
- ✅ Payment processing
- ✅ Complex user interactions
- ✅ Pre-deployment smoke tests

**Tools:**
- **Playwright:** Modern, fast, multi-browser (recommended)
- **Cypress:** Popular, great DX, Chrome-focused
- **Selenium:** Legacy, but supports many browsers
- **Puppeteer:** Chrome/Chromium only

**Example 1: Login flow with Playwright**

```javascript
// tests/e2e/login.spec.js
import { test, expect } from '@playwright/test';

test.describe('Login Flow', () => {
  test('successful login redirects to dashboard', async ({ page }) => {
    // Navigate to login page
    await page.goto('/login');
    
    // Fill in credentials
    await page.getByLabel('Email').fill('user@example.com');
    await page.getByLabel('Password').fill('password123');
    
    // Submit form
    await page.getByRole('button', { name: 'Login' }).click();
    
    // Wait for navigation
    await page.waitForURL('/dashboard');
    
    // Verify we're on dashboard
    await expect(page.getByRole('heading', { name: 'Dashboard' })).toBeVisible();
    
    // Verify user info is displayed
    await expect(page.getByText('Welcome, User')).toBeVisible();
  });
  
  test('failed login shows error message', async ({ page }) => {
    await page.goto('/login');
    
    await page.getByLabel('Email').fill('wrong@example.com');
    await page.getByLabel('Password').fill('wrongpassword');
    await page.getByRole('button', { name: 'Login' }).click();
    
    // Should stay on login page
    await expect(page).toHaveURL('/login');
    
    // Error message appears
    await expect(page.getByRole('alert')).toContainText('Invalid credentials');
  });
  
  test('validates required fields', async ({ page }) => {
    await page.goto('/login');
    
    // Try to submit empty form
    await page.getByRole('button', { name: 'Login' }).click();
    
    // HTML5 validation should prevent submission
    const emailInput = page.getByLabel('Email');
    await expect(emailInput).toHaveAttribute('required');
    
    // Or check for custom validation messages
    await expect(page.getByText('Email is required')).toBeVisible();
  });
});
```

**Example 2: E-commerce checkout with Cypress**

```javascript
// cypress/e2e/checkout.cy.js
describe('Checkout Flow', () => {
  beforeEach(() => {
    // Login before each test
    cy.login('customer@example.com', 'password');
  });
  
  it('completes full checkout process', () => {
    // Add item to cart
    cy.visit('/products/widget-pro');
    cy.get('[data-testid="add-to-cart"]').click();
    
    // Verify cart
    cy.get('[data-testid="cart-count"]').should('contain', '1');
    
    // Go to cart
    cy.get('[data-testid="cart-icon"]').click();
    cy.url().should('include', '/cart');
    
    // Verify item in cart
    cy.contains('Widget Pro').should('be.visible');
    cy.contains('$49.99').should('be.visible');
    
    // Proceed to checkout
    cy.get('[data-testid="checkout-button"]').click();
    
    // Fill shipping info
    cy.get('[name="fullName"]').type('John Doe');
    cy.get('[name="address"]').type('123 Main St');
    cy.get('[name="city"]').type('New York');
    cy.get('[name="zipCode"]').type('10001');
    
    // Continue to payment
    cy.get('[data-testid="continue-to-payment"]').click();
    
    // Fill payment info (test mode)
    cy.get('[name="cardNumber"]').type('4242424242424242');
    cy.get('[name="expiry"]').type('12/25');
    cy.get('[name="cvc"]').type('123');
    
    // Submit order
    cy.get('[data-testid="place-order"]').click();
    
    // Wait for confirmation
    cy.url().should('include', '/order-confirmation');
    cy.contains('Order Confirmed').should('be.visible');
    cy.get('[data-testid="order-number"]').should('exist');
  });
  
  it('prevents checkout with empty cart', () => {
    cy.visit('/cart');
    
    // Empty cart should disable checkout
    cy.get('[data-testid="checkout-button"]').should('be.disabled');
    cy.contains('Your cart is empty').should('be.visible');
  });
  
  it('saves cart state between sessions', () => {
    // Add item
    cy.visit('/products/widget-pro');
    cy.get('[data-testid="add-to-cart"]').click();
    
    // Refresh page
    cy.reload();
    
    // Cart should still have item
    cy.get('[data-testid="cart-count"]').should('contain', '1');
  });
});
```

**Example 3: Multi-page navigation test**

```javascript
// tests/e2e/navigation.spec.js
import { test, expect } from '@playwright/test';

test.describe('Site Navigation', () => {
  test('navigates through main pages', async ({ page }) => {
    await page.goto('/');
    
    // Home page
    await expect(page).toHaveTitle(/Home/);
    
    // Navigate to About
    await page.getByRole('link', { name: 'About' }).click();
    await expect(page).toHaveURL(/\/about/);
    await expect(page.getByRole('heading', { name: 'About Us' })).toBeVisible();
    
    // Navigate to Products
    await page.getByRole('link', { name: 'Products' }).click();
    await expect(page).toHaveURL(/\/products/);
    
    // Click on a product
    await page.getByRole('link', { name: 'Widget Pro' }).click();
    await expect(page).toHaveURL(/\/products\/widget-pro/);
    
    // Breadcrumb navigation
    await page.getByRole('link', { name: 'Products' }).first().click();
    await expect(page).toHaveURL(/\/products/);
  });
  
  test('search functionality works across pages', async ({ page }) => {
    await page.goto('/');
    
    // Use search from home page
    await page.getByPlaceholder('Search...').fill('widget');
    await page.keyboard.press('Enter');
    
    // Should navigate to search results
    await expect(page).toHaveURL(/\/search\?q=widget/);
    await expect(page.getByText('Search results for "widget"')).toBeVisible();
    
    // Search should still work on results page
    await page.getByPlaceholder('Search...').fill('gadget');
    await page.keyboard.press('Enter');
    await expect(page).toHaveURL(/\/search\?q=gadget/);
  });
  
  test('handles 404 pages gracefully', async ({ page }) => {
    await page.goto('/this-page-does-not-exist');
    
    await expect(page.getByText('404')).toBeVisible();
    await expect(page.getByText('Page Not Found')).toBeVisible();
    
    // Can navigate back home
    await page.getByRole('link', { name: 'Go Home' }).click();
    await expect(page).toHaveURL('/');
  });
});
```

**Example 4: Testing with authentication state**

```javascript
// playwright.config.js - Setup authentication
import { test as setup } from '@playwright/test';

setup('authenticate', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Email').fill('user@example.com');
  await page.getByLabel('Password').fill('password123');
  await page.getByRole('button', { name: 'Login' }).click();
  
  // Save auth state
  await page.context().storageState({ path: 'auth.json' });
});

// tests/e2e/authenticated.spec.js - Use saved auth
import { test, expect } from '@playwright/test';

test.use({ storageState: 'auth.json' });

test('can access protected pages when authenticated', async ({ page }) => {
  await page.goto('/profile');
  
  // Should not redirect to login
  await expect(page).toHaveURL('/profile');
  await expect(page.getByRole('heading', { name: 'My Profile' })).toBeVisible();
});
```

**Playwright vs Cypress comparison:**

| Feature | Playwright | Cypress |
|---------|-----------|---------|
| **Browsers** | Chrome, Firefox, Safari, Edge | Chrome, Firefox, Edge |
| **Speed** | Very fast (parallel by default) | Fast |
| **API** | Standard async/await | Custom commands |
| **Network mocking** | Built-in | Built-in |
| **Multiple tabs** | Yes | Limited |
| **Debugging** | VSCode integration, trace viewer | Time-travel debugging |
| **Best for** | Multi-browser, speed, modern apps | DX, simple setup |

**Best Practices:**
- ✅ Test critical user journeys only (E2E tests are slow)
- ✅ Use page object pattern for maintainability
- ✅ Run E2E tests in CI/CD before deployment
- ✅ Test on multiple browsers for compatibility
- ✅ Use data-testid for stable selectors
- ✅ Set up authentication state once, reuse across tests
- ✅ Take screenshots/videos on failure
- ❌ Don't test every edge case (use unit/integration for that)
- ❌ Don't make tests dependent on each other
- ❌ Don't use brittle selectors (class names, XPath)

**When to skip E2E:**
- ❌ Simple static pages
- ❌ Isolated component behavior (use component tests)
- ❌ Unit-level logic (use unit tests)
- ❌ Every possible path (too slow, use integration tests)

---

### 5. Snapshot Tests

**Definition:** Tests that capture the rendered output of a component and compare it against a saved reference snapshot to detect unintended changes.

**Purpose:** Catch unexpected UI changes and regressions in component output.

**When to use:**
- ✅ Stable, mature components
- ✅ Detecting unintentional changes
- ✅ Error messages and static content
- ✅ Complex data structures (JSON, objects)
- ✅ Configuration objects
- ⚠️ Use sparingly for UI (prefer visual regression)

**When NOT to use:**
- ❌ Rapidly changing components
- ❌ Components with dynamic data (dates, IDs)
- ❌ As a replacement for proper assertions
- ❌ Testing behavior (use component tests)

**Tools:**
- **Jest:** Built-in snapshot testing
- **Vitest:** Compatible snapshot testing
- **@testing-library/react:** Render components for snapshots

**Example 1: Basic snapshot test**

```javascript
// ErrorMessage.jsx
export function ErrorMessage({ message, code }) {
  return (
    <div className="error-message" role="alert">
      <strong>Error {code}:</strong> {message}
    </div>
  );
}

// ErrorMessage.test.jsx
import { render } from '@testing-library/react';
import { ErrorMessage } from './ErrorMessage';

describe('ErrorMessage', () => {
  test('matches snapshot', () => {
    const { container } = render(
      <ErrorMessage message="Something went wrong" code={500} />
    );
    
    expect(container).toMatchSnapshot();
  });
});

// Generated snapshot: __snapshots__/ErrorMessage.test.jsx.snap
exports[`ErrorMessage matches snapshot 1`] = `
<div>
  <div
    class="error-message"
    role="alert"
  >
    <strong>
      Error 
      500
      :
    </strong>
     
    Something went wrong
  </div>
</div>
`;
```

**Example 2: Inline snapshots (more readable)**

```javascript
// Badge.jsx
export function Badge({ text, variant = 'default' }) {
  return <span className={`badge badge-${variant}`}>{text}</span>;
}

// Badge.test.jsx
import { render } from '@testing-library/react';
import { Badge } from './Badge';

describe('Badge', () => {
  test('renders default variant', () => {
    const { container } = render(<Badge text="New" />);
    
    expect(container.firstChild).toMatchInlineSnapshot(`
      <span
        class="badge badge-default"
      >
        New
      </span>
    `);
  });
  
  test('renders success variant', () => {
    const { container } = render(<Badge text="Active" variant="success" />);
    
    expect(container.firstChild).toMatchInlineSnapshot(`
      <span
        class="badge badge-success"
      >
        Active
      </span>
    `);
  });
});
```

**Example 3: Snapshot testing data structures**

```javascript
// userFormatter.js
export function formatUserData(user) {
  return {
    id: user.id,
    displayName: `${user.firstName} ${user.lastName}`,
    email: user.email.toLowerCase(),
    roles: user.roles.sort(),
    metadata: {
      createdAt: user.createdAt,
      lastLogin: user.lastLogin,
    },
  };
}

// userFormatter.test.js
import { formatUserData } from './userFormatter';

describe('formatUserData', () => {
  test('formats user data correctly', () => {
    const user = {
      id: 1,
      firstName: 'Alice',
      lastName: 'Smith',
      email: 'ALICE@EXAMPLE.COM',
      roles: ['user', 'admin'],
      createdAt: '2024-01-01',
      lastLogin: '2024-12-19',
    };
    
    const result = formatUserData(user);
    
    // Snapshot is great for complex objects
    expect(result).toMatchInlineSnapshot(`
      {
        "displayName": "Alice Smith",
        "email": "alice@example.com",
        "id": 1,
        "metadata": {
          "createdAt": "2024-01-01",
          "lastLogin": "2024-12-19",
        },
        "roles": [
          "admin",
          "user",
        ],
      }
    `);
  });
});
```

**Example 4: Property matchers for dynamic data**

```javascript
// UserCard.jsx
export function UserCard({ user }) {
  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      <p>ID: {user.id}</p>
      <time>{new Date().toISOString()}</time>
    </div>
  );
}

// UserCard.test.jsx
import { render } from '@testing-library/react';
import { UserCard } from './UserCard';

describe('UserCard', () => {
  test('matches snapshot with dynamic data', () => {
    const user = { id: 123, name: 'Alice' };
    const { container } = render(<UserCard user={user} />);
    
    // Use property matchers for dynamic values
    expect(container.firstChild).toMatchSnapshot({
      // Match any ISO date string
      children: expect.arrayContaining([
        expect.objectContaining({
          type: 'time',
          props: {
            children: expect.stringMatching(/^\d{4}-\d{2}-\d{2}T/),
          },
        }),
      ]),
    });
  });
});
```

**Updating snapshots:**

```bash
# Update all snapshots
npm test -- -u

# Update snapshots interactively
npm test -- --watch

# In watch mode: press 'u' to update failing snapshots
```

**Best Practices:**
- ✅ Keep snapshots small and focused
- ✅ Review snapshot changes carefully in PRs
- ✅ Use inline snapshots for readability
- ✅ Use property matchers for dynamic data
- ✅ Combine with behavior tests (don't rely on snapshots alone)
- ✅ Test stable, public APIs
- ❌ Don't snapshot large components (too brittle)
- ❌ Don't blindly update snapshots without reviewing
- ❌ Don't use as a substitute for proper testing
- ❌ Don't commit snapshots of frequently changing UI

**Snapshot Testing Anti-Patterns:**

```javascript
// ❌ BAD: Snapshotting entire page
test('renders entire app', () => {
  const { container } = render(<App />);
  expect(container).toMatchSnapshot(); // Too large, too brittle
});

// ✅ GOOD: Snapshot small, stable components
test('renders error banner', () => {
  const { container } = render(<ErrorBanner message="Error" />);
  expect(container.firstChild).toMatchSnapshot();
});

// ❌ BAD: Snapshot without understanding
test('whatever this does', () => {
  const { container } = render(<ComplexComponent />);
  expect(container).toMatchSnapshot(); // What are we testing?
});

// ✅ GOOD: Snapshot with clear purpose
test('renders discount badge for premium users', () => {
  const { container } = render(<DiscountBadge isPremium={true} />);
  expect(container.firstChild).toMatchInlineSnapshot(`
    <span class="badge badge-premium">
      20% OFF
    </span>
  `);
});
```

---

### 6. Visual Regression Tests

**Definition:** Tests that capture screenshots of your UI and compare them pixel-by-pixel against baseline images to detect visual changes.

**Purpose:** Catch unintended visual bugs, CSS regressions, and design inconsistencies across browsers and screen sizes.

**When to use:**
- ✅ UI components with complex styling
- ✅ Cross-browser visual consistency
- ✅ Responsive design testing
- ✅ Detecting CSS regressions
- ✅ Theme/brand consistency
- ✅ Animation and transition testing
- ✅ Pre-deployment visual checks

**Tools:**
- **Playwright:** Built-in screenshot comparison
- **Chromatic:** Storybook-based visual testing (recommended)
- **Percy:** Visual testing platform
- **BackstopJS:** Screenshot comparison tool
- **Applitools:** AI-powered visual testing

**Example 1: Playwright visual regression**

```javascript
// tests/visual/button.spec.js
import { test, expect } from '@playwright/test';

test.describe('Button Visual Tests', () => {
  test('primary button matches screenshot', async ({ page }) => {
    await page.goto('/components/buttons');
    
    const button = page.getByRole('button', { name: 'Primary' });
    
    // Take screenshot and compare
    await expect(button).toHaveScreenshot('primary-button.png');
  });
  
  test('button hover state', async ({ page }) => {
    await page.goto('/components/buttons');
    
    const button = page.getByRole('button', { name: 'Primary' });
    
    // Hover and screenshot
    await button.hover();
    await expect(button).toHaveScreenshot('primary-button-hover.png');
  });
  
  test('disabled button state', async ({ page }) => {
    await page.goto('/components/buttons');
    
    const button = page.getByRole('button', { name: 'Disabled' });
    
    await expect(button).toHaveScreenshot('disabled-button.png');
  });
  
  test('buttons across breakpoints', async ({ page }) => {
    await page.goto('/components/buttons');
    
    // Mobile
    await page.setViewportSize({ width: 375, height: 667 });
    await expect(page).toHaveScreenshot('buttons-mobile.png');
    
    // Tablet
    await page.setViewportSize({ width: 768, height: 1024 });
    await expect(page).toHaveScreenshot('buttons-tablet.png');
    
    // Desktop
    await page.setViewportSize({ width: 1920, height: 1080 });
    await expect(page).toHaveScreenshot('buttons-desktop.png');
  });
});
```

**Example 2: Chromatic with Storybook**

```javascript
// Button.stories.jsx
import { Button } from './Button';

export default {
  title: 'Components/Button',
  component: Button,
  parameters: {
    // Chromatic configuration
    chromatic: {
      viewports: [375, 768, 1920], // Test multiple viewports
      delay: 300, // Wait for animations
    },
  },
};

export const Primary = {
  args: {
    variant: 'primary',
    children: 'Click me',
  },
};

export const Secondary = {
  args: {
    variant: 'secondary',
    children: 'Click me',
  },
};

export const Disabled = {
  args: {
    variant: 'primary',
    children: 'Click me',
    disabled: true,
  },
};

export const WithIcon = {
  args: {
    variant: 'primary',
    children: (
      <>
        <IconPlus /> Add Item
      </>
    ),
  },
};

// Interaction states
export const HoverState = {
  args: Primary.args,
  parameters: {
    pseudo: { hover: true },
  },
};

export const FocusState = {
  args: Primary.args,
  parameters: {
    pseudo: { focus: true },
  },
};
```

**Example 3: Percy visual testing**

```javascript
// tests/visual/homepage.spec.js
import percySnapshot from '@percy/playwright';
import { test } from '@playwright/test';

test.describe('Homepage Visual Tests', () => {
  test('homepage renders correctly', async ({ page }) => {
    await page.goto('/');
    
    // Wait for content to load
    await page.waitForSelector('[data-testid="hero-section"]');
    
    // Take Percy snapshot
    await percySnapshot(page, 'Homepage');
  });
  
  test('homepage with dark theme', async ({ page }) => {
    await page.goto('/');
    
    // Toggle dark mode
    await page.getByRole('button', { name: 'Toggle theme' }).click();
    
    await percySnapshot(page, 'Homepage - Dark Theme');
  });
  
  test('homepage responsive', async ({ page }) => {
    await page.goto('/');
    
    // Percy automatically tests multiple viewports
    await percySnapshot(page, 'Homepage - Responsive', {
      widths: [375, 768, 1280, 1920],
    });
  });
  
  test('modal overlay', async ({ page }) => {
    await page.goto('/');
    
    await page.getByRole('button', { name: 'Open Modal' }).click();
    await page.waitForSelector('[role="dialog"]');
    
    await percySnapshot(page, 'Modal Overlay');
  });
});
```

**Example 4: Responsive component testing**

```javascript
// Card.visual.spec.js
import { test, expect } from '@playwright/test';

const viewports = [
  { name: 'mobile', width: 375, height: 667 },
  { name: 'tablet', width: 768, height: 1024 },
  { name: 'desktop', width: 1440, height: 900 },
];

test.describe('Card Component Visual', () => {
  for (const viewport of viewports) {
    test(`card on ${viewport.name}`, async ({ page }) => {
      await page.setViewportSize({ 
        width: viewport.width, 
        height: viewport.height 
      });
      
      await page.goto('/components/card');
      
      const card = page.locator('[data-testid="product-card"]').first();
      
      await expect(card).toHaveScreenshot(
        `card-${viewport.name}.png`,
        {
          maxDiffPixels: 100, // Allow small differences
        }
      );
    });
  }
  
  test('card with long content', async ({ page }) => {
    await page.goto('/components/card?variant=long-content');
    
    const card = page.locator('[data-testid="product-card"]').first();
    
    await expect(card).toHaveScreenshot('card-long-content.png');
  });
  
  test('card loading state', async ({ page }) => {
    await page.goto('/components/card?variant=loading');
    
    const card = page.locator('[data-testid="product-card"]').first();
    
    // Wait for skeleton animation
    await page.waitForTimeout(500);
    
    await expect(card).toHaveScreenshot('card-loading.png', {
      animations: 'disabled', // Disable animations for consistent screenshots
    });
  });
});
```

**Playwright screenshot configuration:**

```javascript
// playwright.config.js
export default {
  expect: {
    toHaveScreenshot: {
      maxDiffPixels: 100,
      threshold: 0.2,
      animations: 'disabled',
    },
  },
  use: {
    screenshot: 'only-on-failure',
  },
};
```

**Updating visual baselines:**

```bash
# Playwright: Update all screenshots
npx playwright test --update-snapshots

# Chromatic: Approve changes in UI
npx chromatic --project-token=<token>

# Percy: Approve in Percy dashboard
# (Changes must be approved through web UI)
```

**Best Practices:**
- ✅ Test key components and critical pages
- ✅ Test multiple viewport sizes
- ✅ Test interactive states (hover, focus, disabled)
- ✅ Disable animations for consistency
- ✅ Test both light and dark themes
- ✅ Use meaningful screenshot names
- ✅ Review visual diffs carefully before approving
- ✅ Run visual tests in CI before deployment
- ❌ Don't screenshot every component (focus on UI-heavy ones)
- ❌ Don't test frequently changing content
- ❌ Don't approve changes without understanding why they occurred

**Visual Testing Strategy:**

```
┌─────────────────────────────────────────┐
│ Development                             │
│ • Run visual tests locally              │
│ • Quick feedback on visual changes      │
└─────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│ Pull Request                            │
│ • Automated visual regression checks    │
│ • Review diffs in PR comments           │
│ • Approve/reject changes                │
└─────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│ CI/CD Pipeline                          │
│ • Run full visual test suite            │
│ • Test multiple browsers & viewports    │
│ • Block deployment on visual failures   │
└─────────────────────────────────────────┘
```

**Tool Comparison:**

| Tool | Best For | Cost | Browser Support |
|------|----------|------|-----------------|
| **Playwright** | Built-in, free, fast | Free | Chrome, Firefox, Safari |
| **Chromatic** | Storybook users | Paid (free tier) | Chromium |
| **Percy** | Enterprise teams | Paid | Chrome, Firefox, Safari |
| **Applitools** | AI-powered testing | Paid | All major browsers |
| **BackstopJS** | Self-hosted | Free | Headless Chrome/Firefox |

---

### 7. Accessibility (a11y) Tests

**Definition:** Tests that verify your application is usable by people with disabilities, including those using screen readers, keyboard navigation, or other assistive technologies.

**Purpose:** Ensure your application meets accessibility standards (WCAG) and is inclusive to all users.

**Why this matters:**
- **Legal compliance:** ADA, Section 508 requirements
- **Ethical responsibility:** 15% of world population has disabilities
- **Better UX for everyone:** Accessible apps are easier to use for all
- **SEO benefits:** Semantic HTML improves search rankings
- **Interviews:** Accessibility knowledge shows you're a thoughtful engineer

**When to test:**
- ✅ Every user-facing component
- ✅ Forms and interactive elements
- ✅ Navigation and menus
- ✅ Modal dialogs and overlays
- ✅ Dynamic content updates
- ✅ Color contrast and visual elements

**Tools:**
- **jest-axe / vitest-axe:** Automated a11y testing in unit tests
- **@axe-core/react:** Runtime accessibility checker
- **eslint-plugin-jsx-a11y:** Lint-time a11y checks
- **@testing-library/react:** Built-in accessibility queries
- **Playwright:** a11y testing in E2E tests
- **pa11y / axe-cli:** Command-line a11y audits

**Example 1: Basic accessibility testing with jest-axe**

```javascript
// Button.jsx
export function Button({ onClick, children, disabled }) {
  return (
    <button onClick={onClick} disabled={disabled} type="button">
      {children}
    </button>
  );
}

// Button.a11y.test.jsx
import { render } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';
import { Button } from './Button';

expect.extend(toHaveNoViolations);

describe('Button Accessibility', () => {
  test('has no accessibility violations', async () => {
    const { container } = render(<Button>Click me</Button>);
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });
  
  test('disabled button has no violations', async () => {
    const { container } = render(<Button disabled>Click me</Button>);
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });
});
```

**Example 2: Testing keyboard navigation**

```javascript
// Dropdown.jsx
export function Dropdown({ options, onSelect }) {
  const [isOpen, setIsOpen] = useState(false);
  const [selectedIndex, setSelectedIndex] = useState(0);
  
  const handleKeyDown = (e) => {
    if (e.key === 'Enter' || e.key === ' ') {
      setIsOpen(!isOpen);
    } else if (e.key === 'ArrowDown' && isOpen) {
      setSelectedIndex((i) => (i + 1) % options.length);
    } else if (e.key === 'ArrowUp' && isOpen) {
      setSelectedIndex((i) => (i - 1 + options.length) % options.length);
    } else if (e.key === 'Escape') {
      setIsOpen(false);
    }
  };
  
  return (
    <div>
      <button
        onClick={() => setIsOpen(!isOpen)}
        onKeyDown={handleKeyDown}
        aria-expanded={isOpen}
        aria-haspopup="listbox"
      >
        Select option
      </button>
      {isOpen && (
        <ul role="listbox">
          {options.map((option, index) => (
            <li
              key={option}
              role="option"
              aria-selected={index === selectedIndex}
              onClick={() => onSelect(option)}
            >
              {option}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}

// Dropdown.a11y.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Dropdown } from './Dropdown';

describe('Dropdown Accessibility', () => {
  const options = ['Option 1', 'Option 2', 'Option 3'];
  
  test('can be opened with keyboard', async () => {
    const user = userEvent.setup();
    render(<Dropdown options={options} onSelect={jest.fn()} />);
    
    const button = screen.getByRole('button');
    
    // Focus and press Enter
    button.focus();
    await user.keyboard('{Enter}');
    
    expect(screen.getByRole('listbox')).toBeInTheDocument();
  });
  
  test('navigates options with arrow keys', async () => {
    const user = userEvent.setup();
    render(<Dropdown options={options} onSelect={jest.fn()} />);
    
    const button = screen.getByRole('button');
    button.focus();
    await user.keyboard('{Enter}');
    
    // Arrow down should select next option
    await user.keyboard('{ArrowDown}');
    
    const secondOption = screen.getByRole('option', { name: 'Option 2' });
    expect(secondOption).toHaveAttribute('aria-selected', 'true');
  });
  
  test('closes with Escape key', async () => {
    const user = userEvent.setup();
    render(<Dropdown options={options} onSelect={jest.fn()} />);
    
    const button = screen.getByRole('button');
    button.focus();
    await user.keyboard('{Enter}');
    
    expect(screen.getByRole('listbox')).toBeInTheDocument();
    
    await user.keyboard('{Escape}');
    
    expect(screen.queryByRole('listbox')).not.toBeInTheDocument();
  });
  
  test('has correct ARIA attributes', () => {
    render(<Dropdown options={options} onSelect={jest.fn()} />);
    
    const button = screen.getByRole('button');
    
    expect(button).toHaveAttribute('aria-haspopup', 'listbox');
    expect(button).toHaveAttribute('aria-expanded', 'false');
  });
});
```

**Example 3: Testing form accessibility**

```javascript
// ContactForm.jsx
export function ContactForm({ onSubmit }) {
  const [errors, setErrors] = useState({});
  
  return (
    <form onSubmit={onSubmit} aria-label="Contact form">
      <div>
        <label htmlFor="name">
          Name <span aria-label="required">*</span>
        </label>
        <input
          id="name"
          type="text"
          required
          aria-required="true"
          aria-invalid={!!errors.name}
          aria-describedby={errors.name ? 'name-error' : undefined}
        />
        {errors.name && (
          <div id="name-error" role="alert">
            {errors.name}
          </div>
        )}
      </div>
      
      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          type="email"
          aria-describedby="email-hint"
        />
        <div id="email-hint" className="hint">
          We'll never share your email
        </div>
      </div>
      
      <button type="submit">Submit</button>
    </form>
  );
}

// ContactForm.a11y.test.jsx
import { render, screen } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';
import { ContactForm } from './ContactForm';

expect.extend(toHaveNoViolations);

describe('ContactForm Accessibility', () => {
  test('has no a11y violations', async () => {
    const { container } = render(<ContactForm onSubmit={jest.fn()} />);
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });
  
  test('labels are properly associated with inputs', () => {
    render(<ContactForm onSubmit={jest.fn()} />);
    
    expect(screen.getByLabelText(/Name/)).toBeInTheDocument();
    expect(screen.getByLabelText('Email')).toBeInTheDocument();
  });
  
  test('required fields are marked', () => {
    render(<ContactForm onSubmit={jest.fn()} />);
    
    const nameInput = screen.getByLabelText(/Name/);
    expect(nameInput).toHaveAttribute('aria-required', 'true');
    expect(nameInput).toBeRequired();
  });
  
  test('error messages are associated with inputs', () => {
    render(<ContactForm onSubmit={jest.fn()} />);
    
    const nameInput = screen.getByLabelText(/Name/);
    
    // Simulate error state
    nameInput.setAttribute('aria-invalid', 'true');
    nameInput.setAttribute('aria-describedby', 'name-error');
    
    expect(nameInput).toHaveAttribute('aria-invalid', 'true');
    expect(nameInput).toHaveAttribute('aria-describedby', 'name-error');
  });
  
  test('helper text is associated with input', () => {
    render(<ContactForm onSubmit={jest.fn()} />);
    
    const emailInput = screen.getByLabelText('Email');
    expect(emailInput).toHaveAttribute('aria-describedby', 'email-hint');
  });
});
```

**Example 4: Testing screen reader announcements**

```javascript
// Notification.jsx
export function Notification({ message, type = 'info' }) {
  return (
    <div
      role="status"
      aria-live={type === 'error' ? 'assertive' : 'polite'}
      aria-atomic="true"
    >
      {message}
    </div>
  );
}

// Notification.a11y.test.jsx
import { render, screen } from '@testing-library/react';
import { Notification } from './Notification';

describe('Notification Accessibility', () => {
  test('info notifications use polite aria-live', () => {
    render(<Notification message="Save successful" type="info" />);
    
    const notification = screen.getByRole('status');
    expect(notification).toHaveAttribute('aria-live', 'polite');
  });
  
  test('error notifications use assertive aria-live', () => {
    render(<Notification message="Error occurred" type="error" />);
    
    const notification = screen.getByRole('status');
    expect(notification).toHaveAttribute('aria-live', 'assertive');
  });
  
  test('has aria-atomic for complete announcements', () => {
    render(<Notification message="Update complete" />);
    
    const notification = screen.getByRole('status');
    expect(notification).toHaveAttribute('aria-atomic', 'true');
  });
});
```

**Example 5: E2E accessibility testing with Playwright**

```javascript
// tests/a11y/homepage.spec.js
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test.describe('Homepage Accessibility', () => {
  test('should not have any automatically detectable a11y issues', async ({ page }) => {
    await page.goto('/');
    
    const accessibilityScanResults = await new AxeBuilder({ page }).analyze();
    
    expect(accessibilityScanResults.violations).toEqual([]);
  });
  
  test('can navigate entire site with keyboard', async ({ page }) => {
    await page.goto('/');
    
    // Tab through all interactive elements
    await page.keyboard.press('Tab');
    let focusedElement = await page.evaluate(() => document.activeElement.tagName);
    expect(['A', 'BUTTON', 'INPUT']).toContain(focusedElement);
    
    // Should be able to reach all main navigation items
    const navLinks = page.locator('nav a');
    const count = await navLinks.count();
    
    for (let i = 0; i < count; i++) {
      await page.keyboard.press('Tab');
    }
  });
  
  test('focus is visible', async ({ page }) => {
    await page.goto('/');
    
    // Tab to first focusable element
    await page.keyboard.press('Tab');
    
    // Check that focus is visible (outline or other indicator)
    const focusedElement = page.locator(':focus');
    await expect(focusedElement).toBeVisible();
  });
});
```

**Common Accessibility Issues:**

| Issue | How to Fix |
|-------|------------|
| **Missing alt text** | Add descriptive `alt` attributes to images |
| **Low contrast** | Use colors with 4.5:1 contrast ratio minimum |
| **No keyboard access** | Ensure all interactive elements are keyboard accessible |
| **Missing labels** | Associate `<label>` with form inputs |
| **Invalid ARIA** | Use correct ARIA roles and attributes |
| **No focus indicators** | Add visible `:focus` styles |
| **Improper heading hierarchy** | Use h1, h2, h3 in order |

**Best Practices:**
- ✅ Test with real screen readers (NVDA, JAWS, VoiceOver)
- ✅ Test keyboard navigation
- ✅ Use semantic HTML (`<button>`, `<nav>`, `<main>`)
- ✅ Provide text alternatives for non-text content
- ✅ Ensure sufficient color contrast
- ✅ Make all functionality keyboard accessible
- ✅ Use ARIA when semantic HTML isn't enough
- ✅ Test with automated tools AND manual testing
- ❌ Don't rely solely on automated testing (catches ~30% of issues)
- ❌ Don't use `div` with click handlers (use `<button>`)
- ❌ Don't hide focus indicators
- ❌ Don't use color as the only indicator

---

### 8. Performance Tests

**Definition:** Tests that measure and verify the performance characteristics of your application, including load times, rendering speed, bundle size, and runtime performance.

**Purpose:** Ensure your application remains fast and responsive as it grows.

**When to test:**
- ✅ Component render performance
- ✅ Large list rendering
- ✅ Expensive calculations
- ✅ Bundle size monitoring
- ✅ Lighthouse scores
- ✅ Core Web Vitals (LCP, FID, CLS)
- ✅ Memory leaks

**Tools:**
- **React DevTools Profiler:** Measure component render times
- **Lighthouse CI:** Automated performance audits
- **bundlesize / size-limit:** Bundle size monitoring
- **@testing-library/react:** `waitFor` performance
- **Playwright:** Performance metrics in E2E tests
- **web-vitals:** Measure Core Web Vitals

**Example 1: Testing component render performance**

```javascript
// LargeList.jsx
import { memo } from 'react';

const ListItem = memo(({ item }) => (
  <div className="list-item">
    <h3>{item.title}</h3>
    <p>{item.description}</p>
  </div>
));

export function LargeList({ items }) {
  return (
    <div>
      {items.map(item => (
        <ListItem key={item.id} item={item} />
      ))}
    </div>
  );
}

// LargeList.perf.test.jsx
import { render } from '@testing-library/react';
import { LargeList } from './LargeList';

describe('LargeList Performance', () => {
  const generateItems = (count) => 
    Array.from({ length: count }, (_, i) => ({
      id: i,
      title: `Item ${i}`,
      description: `Description for item ${i}`,
    }));
  
  test('renders 1000 items in reasonable time', () => {
    const items = generateItems(1000);
    
    const startTime = performance.now();
    render(<LargeList items={items} />);
    const endTime = performance.now();
    
    const renderTime = endTime - startTime;
    
    // Should render in less than 500ms
    expect(renderTime).toBeLessThan(500);
  });
  
  test('re-renders efficiently when items change', () => {
    const items = generateItems(100);
    const { rerender } = render(<LargeList items={items} />);
    
    // Change one item
    const newItems = [...items];
    newItems[0] = { ...newItems[0], title: 'Updated' };
    
    const startTime = performance.now();
    rerender(<LargeList items={newItems} />);
    const endTime = performance.now();
    
    const rerenderTime = endTime - startTime;
    
    // Re-render should be fast (memoization working)
    expect(rerenderTime).toBeLessThan(50);
  });
});
```

**Example 2: Bundle size testing**

```javascript
// size-limit.config.js
module.exports = [
  {
    name: 'Main bundle',
    path: 'dist/index.js',
    limit: '50 KB',
  },
  {
    name: 'Vendor bundle',
    path: 'dist/vendor.js',
    limit: '200 KB',
  },
  {
    name: 'Homepage (First Load JS)',
    path: 'dist/pages/index.js',
    limit: '100 KB',
  },
];

// package.json
{
  "scripts": {
    "size": "size-limit",
    "size:why": "size-limit --why"
  },
  "size-limit": [
    {
      "path": "dist/main.*.js",
      "limit": "50 KB"
    }
  ]
}
```

**Example 3: Lighthouse CI configuration**

```yaml
# .lighthouserc.json
{
  "ci": {
    "collect": {
      "startServerCommand": "npm run start",
      "url": ["http://localhost:3000/"],
      "numberOfRuns": 3
    },
    "assert": {
      "assertions": {
        "categories:performance": ["error", {"minScore": 0.9}],
        "categories:accessibility": ["error", {"minScore": 0.9}],
        "categories:best-practices": ["error", {"minScore": 0.9}],
        "categories:seo": ["error", {"minScore": 0.9}],
        "first-contentful-paint": ["error", {"maxNumericValue": 2000}],
        "largest-contentful-paint": ["error", {"maxNumericValue": 2500}],
        "cumulative-layout-shift": ["error", {"maxNumericValue": 0.1}],
        "total-blocking-time": ["error", {"maxNumericValue": 300}]
      }
    },
    "upload": {
      "target": "temporary-public-storage"
    }
  }
}
```

**Example 4: Testing Core Web Vitals**

```javascript
// tests/performance/web-vitals.spec.js
import { test, expect } from '@playwright/test';

test.describe('Core Web Vitals', () => {
  test('measures LCP (Largest Contentful Paint)', async ({ page }) => {
    await page.goto('/');
    
    const lcp = await page.evaluate(() => {
      return new Promise((resolve) => {
        new PerformanceObserver((list) => {
          const entries = list.getEntries();
          const lastEntry = entries[entries.length - 1];
          resolve(lastEntry.renderTime || lastEntry.loadTime);
        }).observe({ entryTypes: ['largest-contentful-paint'] });
        
        // Timeout after 5 seconds
        setTimeout(() => resolve(null), 5000);
      });
    });
    
    // LCP should be less than 2.5 seconds (good)
    expect(lcp).toBeLessThan(2500);
  });
  
  test('measures FID (First Input Delay)', async ({ page }) => {
    await page.goto('/');
    
    // Click a button
    await page.click('button:first-of-type');
    
    const fid = await page.evaluate(() => {
      return new Promise((resolve) => {
        new PerformanceObserver((list) => {
          const entries = list.getEntries();
          resolve(entries[0]?.processingStart - entries[0]?.startTime);
        }).observe({ entryTypes: ['first-input'] });
      });
    });
    
    // FID should be less than 100ms (good)
    expect(fid).toBeLessThan(100);
  });
  
  test('measures CLS (Cumulative Layout Shift)', async ({ page }) => {
    await page.goto('/');
    
    // Wait for page to stabilize
    await page.waitForLoadState('networkidle');
    
    const cls = await page.evaluate(() => {
      return new Promise((resolve) => {
        let clsScore = 0;
        
        new PerformanceObserver((list) => {
          for (const entry of list.getEntries()) {
            if (!entry.hadRecentInput) {
              clsScore += entry.value;
            }
          }
        }).observe({ entryTypes: ['layout-shift'] });
        
        setTimeout(() => resolve(clsScore), 3000);
      });
    });
    
    // CLS should be less than 0.1 (good)
    expect(cls).toBeLessThan(0.1);
  });
});
```

**Example 5: Memory leak testing**

```javascript
// MemoryLeakTest.test.jsx
import { render, unmount } from '@testing-library/react';
import { ComponentWithSubscription } from './ComponentWithSubscription';

describe('Memory Leak Tests', () => {
  test('cleans up subscriptions on unmount', () => {
    const subscriptionCleanup = jest.fn();
    
    // Mock subscription
    global.addEventListener = jest.fn((event, handler) => {
      return () => subscriptionCleanup();
    });
    
    const { unmount } = render(<ComponentWithSubscription />);
    
    // Unmount component
    unmount();
    
    // Verify cleanup was called
    expect(subscriptionCleanup).toHaveBeenCalled();
  });
  
  test('no memory leaks after multiple mount/unmount cycles', () => {
    const initialMemory = performance.memory?.usedJSHeapSize;
    
    // Mount and unmount 100 times
    for (let i = 0; i < 100; i++) {
      const { unmount } = render(<ComponentWithSubscription />);
      unmount();
    }
    
    // Force garbage collection if available
    if (global.gc) {
      global.gc();
    }
    
    const finalMemory = performance.memory?.usedJSHeapSize;
    const memoryIncrease = finalMemory - initialMemory;
    
    // Memory increase should be minimal (less than 5MB)
    expect(memoryIncrease).toBeLessThan(5 * 1024 * 1024);
  });
});
```

**Example 6: React Profiler API**

```javascript
// ProfiledComponent.test.jsx
import { Profiler } from 'react';
import { render } from '@testing-library/react';
import { ExpensiveComponent } from './ExpensiveComponent';

describe('Component Performance Profiling', () => {
  test('renders without excessive time', () => {
    let renderTime = 0;
    
    const onRender = (id, phase, actualDuration) => {
      renderTime = actualDuration;
    };
    
    render(
      <Profiler id="ExpensiveComponent" onRender={onRender}>
        <ExpensiveComponent data={largeDataset} />
      </Profiler>
    );
    
    // Should render in less than 100ms
    expect(renderTime).toBeLessThan(100);
  });
});
```

**Performance Budget Example:**

```javascript
// performance-budget.js
module.exports = {
  budgets: [
    {
      resourceSizes: [
        { resourceType: 'script', budget: 300 },
        { resourceType: 'stylesheet', budget: 50 },
        { resourceType: 'image', budget: 200 },
        { resourceType: 'total', budget: 500 },
      ],
    },
    {
      timings: [
        { metric: 'first-contentful-paint', budget: 2000 },
        { metric: 'largest-contentful-paint', budget: 2500 },
        { metric: 'time-to-interactive', budget: 3500 },
        { metric: 'cumulative-layout-shift', budget: 0.1 },
      ],
    },
  ],
};
```

**Best Practices:**
- ✅ Set performance budgets and enforce them in CI
- ✅ Monitor bundle size on every PR
- ✅ Test on slow networks and devices
- ✅ Measure Core Web Vitals
- ✅ Profile components with React DevTools
- ✅ Test for memory leaks
- ✅ Use lazy loading for large components
- ✅ Monitor performance in production
- ❌ Don't optimize prematurely
- ❌ Don't ignore mobile performance
- ❌ Don't ship large bundles without code splitting

**Performance Testing Checklist:**

```
□ Bundle size stays under budget
□ Lighthouse score > 90
□ LCP < 2.5s (good)
□ FID < 100ms (good)
□ CLS < 0.1 (good)
□ No memory leaks
□ Fast on slow 3G networks
□ Components render in <100ms
□ No unnecessary re-renders
□ Images are optimized
```

---

### 9. API/Network Tests

**Definition:** Tests that verify API calls, network requests, response handling, error scenarios, and data fetching logic work correctly.

**Purpose:** Ensure your application handles API interactions reliably, including success, error, and edge cases.

**When to use:**
- ✅ REST API calls
- ✅ GraphQL queries and mutations
- ✅ WebSocket connections
- ✅ HTTP error handling (404, 500, timeouts)
- ✅ Request/response interceptors
- ✅ Retry logic and exponential backoff
- ✅ Authentication flows

**Tools:**
- **MSW (Mock Service Worker):** Intercept network requests at network level
- **nock:** HTTP mocking for Node.js
- **jest.mock():** Mock fetch/axios
- **Playwright/Cypress:** Network interception in E2E tests
- **@tanstack/react-query:** Testing with React Query

**Example 1: Testing with MSW (Mock Service Worker)**

```javascript
// api/users.js
export async function fetchUsers() {
  const response = await fetch('/api/users');
  if (!response.ok) throw new Error('Failed to fetch users');
  return response.json();
}

export async function createUser(userData) {
  const response = await fetch('/api/users', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(userData),
  });
  if (!response.ok) throw new Error('Failed to create user');
  return response.json();
}

// api/users.test.js
import { http, HttpResponse } from 'msw';
import { setupServer } from 'msw/node';
import { fetchUsers, createUser } from './users';

const server = setupServer(
  http.get('/api/users', () => {
    return HttpResponse.json([
      { id: 1, name: 'Alice' },
      { id: 2, name: 'Bob' },
    ]);
  }),
  
  http.post('/api/users', async ({ request }) => {
    const body = await request.json();
    return HttpResponse.json({ id: 3, ...body });
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

describe('API Tests', () => {
  test('fetchUsers returns user list', async () => {
    const users = await fetchUsers();
    
    expect(users).toHaveLength(2);
    expect(users[0].name).toBe('Alice');
  });
  
  test('createUser sends correct data', async () => {
    const newUser = { name: 'Charlie', email: 'charlie@example.com' };
    const result = await createUser(newUser);
    
    expect(result).toEqual({
      id: 3,
      name: 'Charlie',
      email: 'charlie@example.com',
    });
  });
  
  test('fetchUsers handles 500 error', async () => {
    server.use(
      http.get('/api/users', () => {
        return new HttpResponse(null, { status: 500 });
      })
    );
    
    await expect(fetchUsers()).rejects.toThrow('Failed to fetch users');
  });
  
  test('handles network timeout', async () => {
    server.use(
      http.get('/api/users', async () => {
        await new Promise(resolve => setTimeout(resolve, 10000));
        return HttpResponse.json([]);
      })
    );
    
    // Set shorter timeout for test
    await expect(
      Promise.race([
        fetchUsers(),
        new Promise((_, reject) => 
          setTimeout(() => reject(new Error('Timeout')), 1000)
        ),
      ])
    ).rejects.toThrow('Timeout');
  });
});
```

**Example 2: Testing React Query with MSW**

```javascript
// hooks/useUsers.js
import { useQuery } from '@tanstack/react-query';

export function useUsers() {
  return useQuery({
    queryKey: ['users'],
    queryFn: async () => {
      const response = await fetch('/api/users');
      if (!response.ok) throw new Error('Failed to fetch');
      return response.json();
    },
  });
}

// hooks/useUsers.test.jsx
import { renderHook, waitFor } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { http, HttpResponse } from 'msw';
import { setupServer } from 'msw/node';
import { useUsers } from './useUsers';

const server = setupServer(
  http.get('/api/users', () => {
    return HttpResponse.json([
      { id: 1, name: 'Alice' },
      { id: 2, name: 'Bob' },
    ]);
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

function createWrapper() {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false },
    },
  });
  
  return ({ children }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
}

describe('useUsers', () => {
  test('fetches and returns users', async () => {
    const { result } = renderHook(() => useUsers(), {
      wrapper: createWrapper(),
    });
    
    expect(result.current.isLoading).toBe(true);
    
    await waitFor(() => expect(result.current.isSuccess).toBe(true));
    
    expect(result.current.data).toHaveLength(2);
    expect(result.current.data[0].name).toBe('Alice');
  });
  
  test('handles API errors', async () => {
    server.use(
      http.get('/api/users', () => {
        return new HttpResponse(null, { status: 500 });
      })
    );
    
    const { result } = renderHook(() => useUsers(), {
      wrapper: createWrapper(),
    });
    
    await waitFor(() => expect(result.current.isError).toBe(true));
    
    expect(result.current.error).toBeDefined();
  });
});
```

**Example 3: Testing GraphQL with MSW**

```javascript
// graphql/queries.js
import { gql } from '@apollo/client';

export const GET_USERS = gql`
  query GetUsers {
    users {
      id
      name
      email
    }
  }
`;

// hooks/useUsersQuery.test.jsx
import { MockedProvider } from '@apollo/client/testing';
import { renderHook, waitFor } from '@testing-library/react';
import { useQuery } from '@apollo/client';
import { GET_USERS } from '../graphql/queries';

const mocks = [
  {
    request: {
      query: GET_USERS,
    },
    result: {
      data: {
        users: [
          { id: 1, name: 'Alice', email: 'alice@example.com' },
          { id: 2, name: 'Bob', email: 'bob@example.com' },
        ],
      },
    },
  },
];

const errorMocks = [
  {
    request: {
      query: GET_USERS,
    },
    error: new Error('Network error'),
  },
];

describe('GraphQL Query Tests', () => {
  test('fetches users successfully', async () => {
    const { result } = renderHook(
      () => useQuery(GET_USERS),
      {
        wrapper: ({ children }) => (
          <MockedProvider mocks={mocks}>
            {children}
          </MockedProvider>
        ),
      }
    );
    
    expect(result.current.loading).toBe(true);
    
    await waitFor(() => expect(result.current.loading).toBe(false));
    
    expect(result.current.data.users).toHaveLength(2);
    expect(result.current.data.users[0].name).toBe('Alice');
  });
  
  test('handles GraphQL errors', async () => {
    const { result } = renderHook(
      () => useQuery(GET_USERS),
      {
        wrapper: ({ children }) => (
          <MockedProvider mocks={errorMocks}>
            {children}
          </MockedProvider>
        ),
      }
    );
    
    await waitFor(() => expect(result.current.error).toBeDefined());
    
    expect(result.current.error.message).toBe('Network error');
  });
});
```

**Example 4: Testing retry logic**

```javascript
// api/fetchWithRetry.js
export async function fetchWithRetry(url, options = {}, maxRetries = 3) {
  let lastError;
  
  for (let i = 0; i < maxRetries; i++) {
    try {
      const response = await fetch(url, options);
      if (response.ok) return response.json();
      
      // Don't retry client errors (4xx)
      if (response.status >= 400 && response.status < 500) {
        throw new Error(`Client error: ${response.status}`);
      }
      
      lastError = new Error(`Server error: ${response.status}`);
    } catch (error) {
      lastError = error;
    }
    
    // Wait before retrying (exponential backoff)
    if (i < maxRetries - 1) {
      await new Promise(resolve => setTimeout(resolve, 2 ** i * 100));
    }
  }
  
  throw lastError;
}

// api/fetchWithRetry.test.js
import { http, HttpResponse } from 'msw';
import { setupServer } from 'msw/node';
import { fetchWithRetry } from './fetchWithRetry';

const server = setupServer();

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

describe('fetchWithRetry', () => {
  test('succeeds on first try', async () => {
    server.use(
      http.get('/api/data', () => {
        return HttpResponse.json({ success: true });
      })
    );
    
    const result = await fetchWithRetry('/api/data');
    expect(result).toEqual({ success: true });
  });
  
  test('retries on 500 error and eventually succeeds', async () => {
    let attempts = 0;
    
    server.use(
      http.get('/api/data', () => {
        attempts++;
        if (attempts < 3) {
          return new HttpResponse(null, { status: 500 });
        }
        return HttpResponse.json({ success: true });
      })
    );
    
    const result = await fetchWithRetry('/api/data');
    
    expect(attempts).toBe(3);
    expect(result).toEqual({ success: true });
  });
  
  test('does not retry on 404', async () => {
    let attempts = 0;
    
    server.use(
      http.get('/api/data', () => {
        attempts++;
        return new HttpResponse(null, { status: 404 });
      })
    );
    
    await expect(fetchWithRetry('/api/data')).rejects.toThrow('Client error: 404');
    expect(attempts).toBe(1);
  });
  
  test('gives up after max retries', async () => {
    server.use(
      http.get('/api/data', () => {
        return new HttpResponse(null, { status: 500 });
      })
    );
    
    await expect(fetchWithRetry('/api/data', {}, 3)).rejects.toThrow('Server error: 500');
  });
});
```

**Best Practices:**
- ✅ Mock at the network level (MSW) for realistic tests
- ✅ Test success, error, and timeout scenarios
- ✅ Verify request headers and body
- ✅ Test retry and exponential backoff logic
- ✅ Test loading and error states
- ✅ Mock realistic response delays
- ❌ Don't make real API calls in tests
- ❌ Don't test the API itself (focus on your code)
- ❌ Don't ignore error cases

---

### 10. State Management Tests

**Definition:** Tests that verify state management logic, including Redux/Zustand stores, reducers, selectors, and state updates.

**Purpose:** Ensure application state is managed correctly and state changes produce expected results.

**When to use:**
- ✅ Redux reducers and actions
- ✅ Zustand/Jotai stores
- ✅ Context providers
- ✅ State selectors
- ✅ Middleware and side effects
- ✅ Optimistic updates
- ✅ Undo/redo functionality

**Tools:**
- **Redux:** `@reduxjs/toolkit/testing`
- **Zustand:** Direct store testing
- **React Testing Library:** Context testing
- **jest:** Test runner

**Example 1: Testing Redux Toolkit slice**

```javascript
// store/counterSlice.js
import { createSlice } from '@reduxjs/toolkit';

const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment: (state) => {
      state.value += 1;
    },
    decrement: (state) => {
      state.value -= 1;
    },
    incrementByAmount: (state, action) => {
      state.value += action.payload;
    },
    reset: (state) => {
      state.value = 0;
    },
  },
});

export const { increment, decrement, incrementByAmount, reset } = counterSlice.actions;
export default counterSlice.reducer;

// store/counterSlice.test.js
import counterReducer, { increment, decrement, incrementByAmount, reset } from './counterSlice';

describe('counter reducer', () => {
  const initialState = { value: 0 };
  
  test('should handle initial state', () => {
    expect(counterReducer(undefined, { type: 'unknown' })).toEqual({ value: 0 });
  });
  
  test('should handle increment', () => {
    const actual = counterReducer(initialState, increment());
    expect(actual.value).toEqual(1);
  });
  
  test('should handle decrement', () => {
    const actual = counterReducer({ value: 1 }, decrement());
    expect(actual.value).toEqual(0);
  });
  
  test('should handle incrementByAmount', () => {
    const actual = counterReducer(initialState, incrementByAmount(5));
    expect(actual.value).toEqual(5);
  });
  
  test('should handle reset', () => {
    const actual = counterReducer({ value: 10 }, reset());
    expect(actual.value).toEqual(0);
  });
});
```

**Example 2: Testing Redux selectors**

```javascript
// store/selectors.js
export const selectCounter = (state) => state.counter.value;
export const selectIsPositive = (state) => state.counter.value > 0;
export const selectDoubleValue = (state) => state.counter.value * 2;

// store/selectors.test.js
import { selectCounter, selectIsPositive, selectDoubleValue } from './selectors';

describe('counter selectors', () => {
  const state = {
    counter: { value: 5 },
  };
  
  test('selectCounter returns counter value', () => {
    expect(selectCounter(state)).toBe(5);
  });
  
  test('selectIsPositive returns true for positive values', () => {
    expect(selectIsPositive(state)).toBe(true);
    expect(selectIsPositive({ counter: { value: -1 } })).toBe(false);
  });
  
  test('selectDoubleValue returns doubled value', () => {
    expect(selectDoubleValue(state)).toBe(10);
  });
});
```

**Example 3: Testing Zustand store**

```javascript
// store/useCartStore.js
import { create } from 'zustand';

export const useCartStore = create((set) => ({
  items: [],
  
  addItem: (item) => set((state) => ({
    items: [...state.items, item],
  })),
  
  removeItem: (id) => set((state) => ({
    items: state.items.filter(item => item.id !== id),
  })),
  
  clearCart: () => set({ items: [] }),
  
  getTotal: (state) => state.items.reduce((sum, item) => sum + item.price, 0),
}));

// store/useCartStore.test.js
import { act, renderHook } from '@testing-library/react';
import { useCartStore } from './useCartStore';

describe('useCartStore', () => {
  beforeEach(() => {
    // Reset store before each test
    useCartStore.setState({ items: [] });
  });
  
  test('adds item to cart', () => {
    const { result } = renderHook(() => useCartStore());
    
    act(() => {
      result.current.addItem({ id: 1, name: 'Widget', price: 10 });
    });
    
    expect(result.current.items).toHaveLength(1);
    expect(result.current.items[0].name).toBe('Widget');
  });
  
  test('removes item from cart', () => {
    const { result } = renderHook(() => useCartStore());
    
    act(() => {
      result.current.addItem({ id: 1, name: 'Widget', price: 10 });
      result.current.addItem({ id: 2, name: 'Gadget', price: 20 });
    });
    
    act(() => {
      result.current.removeItem(1);
    });
    
    expect(result.current.items).toHaveLength(1);
    expect(result.current.items[0].id).toBe(2);
  });
  
  test('clears cart', () => {
    const { result } = renderHook(() => useCartStore());
    
    act(() => {
      result.current.addItem({ id: 1, name: 'Widget', price: 10 });
      result.current.clearCart();
    });
    
    expect(result.current.items).toHaveLength(0);
  });
  
  test('calculates total', () => {
    const { result } = renderHook(() => useCartStore());
    
    act(() => {
      result.current.addItem({ id: 1, name: 'Widget', price: 10 });
      result.current.addItem({ id: 2, name: 'Gadget', price: 20 });
    });
    
    const total = result.current.getTotal(result.current);
    expect(total).toBe(30);
  });
});
```

**Example 4: Testing Redux async thunks**

```javascript
// store/userSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

export const fetchUser = createAsyncThunk(
  'user/fetchUser',
  async (userId) => {
    const response = await fetch(`/api/users/${userId}`);
    return response.json();
  }
);

const userSlice = createSlice({
  name: 'user',
  initialState: {
    data: null,
    status: 'idle',
    error: null,
  },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchUser.pending, (state) => {
        state.status = 'loading';
      })
      .addCase(fetchUser.fulfilled, (state, action) => {
        state.status = 'succeeded';
        state.data = action.payload;
      })
      .addCase(fetchUser.rejected, (state, action) => {
        state.status = 'failed';
        state.error = action.error.message;
      });
  },
});

export default userSlice.reducer;

// store/userSlice.test.js
import { configureStore } from '@reduxjs/toolkit';
import { http, HttpResponse } from 'msw';
import { setupServer } from 'msw/node';
import userReducer, { fetchUser } from './userSlice';

const server = setupServer(
  http.get('/api/users/:userId', ({ params }) => {
    return HttpResponse.json({
      id: params.userId,
      name: 'Alice',
    });
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

describe('user slice with async thunks', () => {
  let store;
  
  beforeEach(() => {
    store = configureStore({
      reducer: {
        user: userReducer,
      },
    });
  });
  
  test('fetches user successfully', async () => {
    await store.dispatch(fetchUser('1'));
    
    const state = store.getState().user;
    
    expect(state.status).toBe('succeeded');
    expect(state.data.name).toBe('Alice');
    expect(state.error).toBe(null);
  });
  
  test('handles fetch error', async () => {
    server.use(
      http.get('/api/users/:userId', () => {
        return new HttpResponse(null, { status: 500 });
      })
    );
    
    await store.dispatch(fetchUser('1'));
    
    const state = store.getState().user;
    
    expect(state.status).toBe('failed');
    expect(state.error).toBeDefined();
  });
  
  test('sets loading status while fetching', () => {
    store.dispatch(fetchUser('1'));
    
    const state = store.getState().user;
    expect(state.status).toBe('loading');
  });
});
```

**Best Practices:**
- ✅ Test reducers as pure functions
- ✅ Test selectors independently
- ✅ Reset store state between tests
- ✅ Test async actions with MSW
- ✅ Verify state transitions
- ✅ Test edge cases and error states
- ❌ Don't test Redux/Zustand library code
- ❌ Don't test implementation details

---

### 11. Router/Navigation Tests

**Definition:** Tests that verify routing logic, navigation flows, route guards, and URL parameter handling work correctly.

**Purpose:** Ensure users can navigate through your application as expected.

**When to use:**
- ✅ React Router navigation
- ✅ Dynamic routes with parameters
- ✅ Protected/authenticated routes
- ✅ 404 handling
- ✅ Redirects
- ✅ Query parameters
- ✅ Programmatic navigation

**Tools:**
- **React Router:** `react-router-dom`
- **React Testing Library:** `MemoryRouter` for tests
- **Playwright/Cypress:** E2E navigation testing

**Example 1: Testing basic routing**

```javascript
// App.jsx
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

function Home() {
  return <h1>Home Page</h1>;
}

function About() {
  return <h1>About Page</h1>;
}

function NotFound() {
  return <h1>404 Not Found</h1>;
}

export function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
      </nav>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}

// App.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { MemoryRouter } from 'react-router-dom';
import { App } from './App';

describe('App Routing', () => {
  test('renders home page by default', () => {
    render(
      <MemoryRouter initialEntries={['/']}>
        <App />
      </MemoryRouter>
    );
    
    expect(screen.getByText('Home Page')).toBeInTheDocument();
  });
  
  test('navigates to about page', async () => {
    const user = userEvent.setup();
    render(<App />);
    
    await user.click(screen.getByRole('link', { name: 'About' }));
    
    expect(screen.getByText('About Page')).toBeInTheDocument();
  });
  
  test('shows 404 for unknown routes', () => {
    render(
      <MemoryRouter initialEntries={['/unknown']}>
        <App />
      </MemoryRouter>
    );
    
    expect(screen.getByText('404 Not Found')).toBeInTheDocument();
  });
});
```

**Example 2: Testing dynamic routes with parameters**

```javascript
// UserProfile.jsx
import { useParams } from 'react-router-dom';

export function UserProfile() {
  const { userId } = useParams();
  
  return (
    <div>
      <h1>User Profile</h1>
      <p>User ID: {userId}</p>
    </div>
  );
}

// UserProfile.test.jsx
import { render, screen } from '@testing-library/react';
import { MemoryRouter, Route, Routes } from 'react-router-dom';
import { UserProfile } from './UserProfile';

describe('UserProfile', () => {
  test('displays user ID from URL params', () => {
    render(
      <MemoryRouter initialEntries={['/users/123']}>
        <Routes>
          <Route path="/users/:userId" element={<UserProfile />} />
        </Routes>
      </MemoryRouter>
    );
    
    expect(screen.getByText('User ID: 123')).toBeInTheDocument();
  });
});
```

**Example 3: Testing protected routes**

```javascript
// ProtectedRoute.jsx
import { Navigate } from 'react-router-dom';
import { useAuth } from './AuthContext';

export function ProtectedRoute({ children }) {
  const { user } = useAuth();
  
  if (!user) {
    return <Navigate to="/login" replace />;
  }
  
  return children;
}

// ProtectedRoute.test.jsx
import { render, screen } from '@testing-library/react';
import { MemoryRouter, Routes, Route } from 'react-router-dom';
import { AuthProvider } from './AuthContext';
import { ProtectedRoute } from './ProtectedRoute';

function Dashboard() {
  return <h1>Dashboard</h1>;
}

function Login() {
  return <h1>Login Page</h1>;
}

describe('ProtectedRoute', () => {
  test('redirects to login when not authenticated', () => {
    render(
      <MemoryRouter initialEntries={['/dashboard']}>
        <AuthProvider value={{ user: null }}>
          <Routes>
            <Route path="/login" element={<Login />} />
            <Route
              path="/dashboard"
              element={
                <ProtectedRoute>
                  <Dashboard />
                </ProtectedRoute>
              }
            />
          </Routes>
        </AuthProvider>
      </MemoryRouter>
    );
    
    expect(screen.getByText('Login Page')).toBeInTheDocument();
  });
  
  test('renders protected content when authenticated', () => {
    render(
      <MemoryRouter initialEntries={['/dashboard']}>
        <AuthProvider value={{ user: { id: 1, name: 'Alice' } }}>
          <Routes>
            <Route path="/login" element={<Login />} />
            <Route
              path="/dashboard"
              element={
                <ProtectedRoute>
                  <Dashboard />
                </ProtectedRoute>
              }
            />
          </Routes>
        </AuthProvider>
      </MemoryRouter>
    );
    
    expect(screen.getByText('Dashboard')).toBeInTheDocument();
  });
});
```

**Example 4: Testing programmatic navigation**

```javascript
// LoginForm.jsx
import { useNavigate } from 'react-router-dom';

export function LoginForm() {
  const navigate = useNavigate();
  
  const handleSubmit = (e) => {
    e.preventDefault();
    // Simulate successful login
    navigate('/dashboard');
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <button type="submit">Login</button>
    </form>
  );
}

// LoginForm.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { MemoryRouter, Routes, Route } from 'react-router-dom';
import { LoginForm } from './LoginForm';

function Dashboard() {
  return <h1>Dashboard</h1>;
}

describe('LoginForm Navigation', () => {
  test('navigates to dashboard on successful login', async () => {
    const user = userEvent.setup();
    
    render(
      <MemoryRouter initialEntries={['/login']}>
        <Routes>
          <Route path="/login" element={<LoginForm />} />
          <Route path="/dashboard" element={<Dashboard />} />
        </Routes>
      </MemoryRouter>
    );
    
    await user.click(screen.getByRole('button', { name: 'Login' }));
    
    expect(screen.getByText('Dashboard')).toBeInTheDocument();
  });
});
```

**Example 5: Testing query parameters**

```javascript
// SearchResults.jsx
import { useSearchParams } from 'react-router-dom';

export function SearchResults() {
  const [searchParams] = useSearchParams();
  const query = searchParams.get('q');
  const filter = searchParams.get('filter') || 'all';
  
  return (
    <div>
      <h1>Search Results</h1>
      <p>Query: {query}</p>
      <p>Filter: {filter}</p>
    </div>
  );
}

// SearchResults.test.jsx
import { render, screen } from '@testing-library/react';
import { MemoryRouter, Route, Routes } from 'react-router-dom';
import { SearchResults } from './SearchResults';

describe('SearchResults', () => {
  test('displays query from URL params', () => {
    render(
      <MemoryRouter initialEntries={['/search?q=react']}>
        <Routes>
          <Route path="/search" element={<SearchResults />} />
        </Routes>
      </MemoryRouter>
    );
    
    expect(screen.getByText('Query: react')).toBeInTheDocument();
    expect(screen.getByText('Filter: all')).toBeInTheDocument();
  });
  
  test('displays custom filter', () => {
    render(
      <MemoryRouter initialEntries={['/search?q=react&filter=recent']}>
        <Routes>
          <Route path="/search" element={<SearchResults />} />
        </Routes>
      </MemoryRouter>
    );
    
    expect(screen.getByText('Query: react')).toBeInTheDocument();
    expect(screen.getByText('Filter: recent')).toBeInTheDocument();
  });
});
```

**Best Practices:**
- ✅ Use MemoryRouter for isolated tests
- ✅ Test navigation flows
- ✅ Test route guards and redirects
- ✅ Verify URL parameters are handled correctly
- ✅ Test 404 and error routes
- ✅ Test programmatic navigation
- ❌ Don't test React Router library code
- ❌ Don't use BrowserRouter in unit tests

---

### 12. Security Tests

**Definition:** Tests that verify your application handles security concerns correctly, including XSS, CSRF, authentication, authorization, and input validation.

**Purpose:** Prevent security vulnerabilities and ensure sensitive data is protected.

**When to use:**
- ✅ Input sanitization
- ✅ XSS (Cross-Site Scripting) prevention
- ✅ CSRF (Cross-Site Request Forgery) protection
- ✅ Authentication flows
- ✅ Authorization and permissions
- ✅ Sensitive data handling
- ✅ Content Security Policy (CSP)
- ✅ Secure headers

**Tools:**
- **jest-dom:** HTML sanitization checks
- **DOMPurify:** HTML sanitization library
- **Playwright/Cypress:** E2E security testing
- **OWASP ZAP:** Security scanning
- **npm audit:** Dependency vulnerability scanning

**Example 1: Testing XSS prevention**

```javascript
// SafeHTML.jsx
import DOMPurify from 'dompurify';

export function SafeHTML({ html }) {
  const sanitized = DOMPurify.sanitize(html);
  
  return <div dangerouslySetInnerHTML={{ __html: sanitized }} />;
}

// SafeHTML.test.jsx
import { render, screen } from '@testing-library/react';
import { SafeHTML } from './SafeHTML';

describe('SafeHTML XSS Prevention', () => {
  test('renders safe HTML content', () => {
    const safeHTML = '<p>Hello <strong>World</strong></p>';
    render(<SafeHTML html={safeHTML} />);
    
    expect(screen.getByText(/Hello/)).toBeInTheDocument();
    expect(screen.getByText('World')).toBeInTheDocument();
  });
  
  test('removes script tags', () => {
    const maliciousHTML = '<p>Hello</p><script>alert("XSS")</script>';
    const { container } = render(<SafeHTML html={maliciousHTML} />);
    
    // Script should be removed
    expect(container.querySelector('script')).toBeNull();
    expect(screen.getByText('Hello')).toBeInTheDocument();
  });
  
  test('removes onclick handlers', () => {
    const maliciousHTML = '<button onclick="alert(\'XSS\')">Click me</button>';
    const { container } = render(<SafeHTML html={maliciousHTML} />);
    
    const button = container.querySelector('button');
    expect(button).toBeInTheDocument();
    expect(button.getAttribute('onclick')).toBeNull();
  });
  
  test('removes javascript: URLs', () => {
    const maliciousHTML = '<a href="javascript:alert(\'XSS\')">Click</a>';
    const { container } = render(<SafeHTML html={maliciousHTML} />);
    
    const link = container.querySelector('a');
    expect(link.getAttribute('href')).toBeNull();
  });
  
  test('allows data attributes but sanitizes values', () => {
    const html = '<div data-id="123" data-hack="<script>alert(\'XSS\')</script>">Safe</div>';
    const { container } = render(<SafeHTML html={html} />);
    
    const div = container.querySelector('div');
    expect(div.getAttribute('data-id')).toBe('123');
    // DOMPurify should sanitize the data attribute value
    expect(div.getAttribute('data-hack')).not.toContain('<script>');
  });
});
```

**Example 2: Testing input validation and sanitization**

```javascript
// CommentForm.jsx
export function CommentForm({ onSubmit }) {
  const [comment, setComment] = useState('');
  const [error, setError] = useState('');
  
  const validateComment = (text) => {
    // No HTML tags allowed
    if (/<[^>]*>/g.test(text)) {
      return 'HTML tags are not allowed';
    }
    
    // Max length
    if (text.length > 500) {
      return 'Comment must be 500 characters or less';
    }
    
    // No suspicious patterns
    if (/javascript:|on\w+=/i.test(text)) {
      return 'Invalid content detected';
    }
    
    return null;
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    
    const validationError = validateComment(comment);
    if (validationError) {
      setError(validationError);
      return;
    }
    
    onSubmit(comment);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <textarea
        value={comment}
        onChange={(e) => setComment(e.target.value)}
        placeholder="Enter your comment"
      />
      {error && <div role="alert">{error}</div>}
      <button type="submit">Submit</button>
    </form>
  );
}

// CommentForm.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { CommentForm } from './CommentForm';

describe('CommentForm Security', () => {
  test('rejects HTML tags', async () => {
    const user = userEvent.setup();
    const onSubmit = jest.fn();
    render(<CommentForm onSubmit={onSubmit} />);
    
    await user.type(screen.getByPlaceholderText('Enter your comment'), '<script>alert("XSS")</script>');
    await user.click(screen.getByRole('button', { name: 'Submit' }));
    
    expect(screen.getByRole('alert')).toHaveTextContent('HTML tags are not allowed');
    expect(onSubmit).not.toHaveBeenCalled();
  });
  
  test('rejects javascript: URLs', async () => {
    const user = userEvent.setup();
    const onSubmit = jest.fn();
    render(<CommentForm onSubmit={onSubmit} />);
    
    await user.type(screen.getByPlaceholderText('Enter your comment'), 'Check this: javascript:alert("XSS")');
    await user.click(screen.getByRole('button', { name: 'Submit' }));
    
    expect(screen.getByRole('alert')).toHaveTextContent('Invalid content detected');
    expect(onSubmit).not.toHaveBeenCalled();
  });
  
  test('rejects event handlers', async () => {
    const user = userEvent.setup();
    const onSubmit = jest.fn();
    render(<CommentForm onSubmit={onSubmit} />);
    
    await user.type(screen.getByPlaceholderText('Enter your comment'), 'onclick=alert("XSS")');
    await user.click(screen.getByRole('button', { name: 'Submit' }));
    
    expect(screen.getByRole('alert')).toHaveTextContent('Invalid content detected');
    expect(onSubmit).not.toHaveBeenCalled();
  });
  
  test('enforces max length', async () => {
    const user = userEvent.setup();
    const onSubmit = jest.fn();
    render(<CommentForm onSubmit={onSubmit} />);
    
    const longText = 'a'.repeat(501);
    await user.type(screen.getByPlaceholderText('Enter your comment'), longText);
    await user.click(screen.getByRole('button', { name: 'Submit' }));
    
    expect(screen.getByRole('alert')).toHaveTextContent('Comment must be 500 characters or less');
    expect(onSubmit).not.toHaveBeenCalled();
  });
  
  test('accepts safe content', async () => {
    const user = userEvent.setup();
    const onSubmit = jest.fn();
    render(<CommentForm onSubmit={onSubmit} />);
    
    await user.type(screen.getByPlaceholderText('Enter your comment'), 'This is a safe comment!');
    await user.click(screen.getByRole('button', { name: 'Submit' }));
    
    expect(onSubmit).toHaveBeenCalledWith('This is a safe comment!');
    expect(screen.queryByRole('alert')).not.toBeInTheDocument();
  });
});
```

**Example 3: Testing authentication and authorization**

```javascript
// AdminPanel.jsx
import { useAuth } from './AuthContext';

export function AdminPanel() {
  const { user, hasRole } = useAuth();
  
  if (!user) {
    return <div>Please log in</div>;
  }
  
  if (!hasRole('admin')) {
    return <div role="alert">Access denied. Admin privileges required.</div>;
  }
  
  return (
    <div>
      <h1>Admin Panel</h1>
      <button>Delete User</button>
      <button>View Logs</button>
    </div>
  );
}

// AdminPanel.test.jsx
import { render, screen } from '@testing-library/react';
import { AuthProvider } from './AuthContext';
import { AdminPanel } from './AdminPanel';

describe('AdminPanel Authorization', () => {
  test('shows login message when not authenticated', () => {
    render(
      <AuthProvider value={{ user: null, hasRole: () => false }}>
        <AdminPanel />
      </AuthProvider>
    );
    
    expect(screen.getByText('Please log in')).toBeInTheDocument();
    expect(screen.queryByText('Admin Panel')).not.toBeInTheDocument();
  });
  
  test('shows access denied for non-admin users', () => {
    const user = { id: 1, name: 'Alice', roles: ['user'] };
    const hasRole = (role) => user.roles.includes(role);
    
    render(
      <AuthProvider value={{ user, hasRole }}>
        <AdminPanel />
      </AuthProvider>
    );
    
    expect(screen.getByRole('alert')).toHaveTextContent('Access denied');
    expect(screen.queryByText('Admin Panel')).not.toBeInTheDocument();
  });
  
  test('renders admin panel for admin users', () => {
    const user = { id: 1, name: 'Admin', roles: ['admin'] };
    const hasRole = (role) => user.roles.includes(role);
    
    render(
      <AuthProvider value={{ user, hasRole }}>
        <AdminPanel />
      </AuthProvider>
    );
    
    expect(screen.getByText('Admin Panel')).toBeInTheDocument();
    expect(screen.getByRole('button', { name: 'Delete User' })).toBeInTheDocument();
  });
  
  test('does not expose admin actions to regular users via DOM', () => {
    const user = { id: 1, name: 'Alice', roles: ['user'] };
    const hasRole = (role) => user.roles.includes(role);
    
    const { container } = render(
      <AuthProvider value={{ user, hasRole }}>
        <AdminPanel />
      </AuthProvider>
    );
    
    // Ensure no admin buttons are rendered (not just hidden)
    expect(container.querySelector('button')).toBeNull();
  });
});
```

**Example 4: Testing CSRF token handling**

```javascript
// api/secureRequest.js
export async function securePost(url, data) {
  const csrfToken = document.querySelector('meta[name="csrf-token"]')?.content;
  
  if (!csrfToken) {
    throw new Error('CSRF token not found');
  }
  
  const response = await fetch(url, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-CSRF-Token': csrfToken,
    },
    body: JSON.stringify(data),
  });
  
  return response.json();
}

// api/secureRequest.test.js
import { securePost } from './secureRequest';

describe('CSRF Protection', () => {
  beforeEach(() => {
    // Clean up any existing meta tags
    document.querySelectorAll('meta[name="csrf-token"]').forEach(el => el.remove());
  });
  
  test('includes CSRF token in request headers', async () => {
    // Setup CSRF token in DOM
    const meta = document.createElement('meta');
    meta.name = 'csrf-token';
    meta.content = 'test-csrf-token-123';
    document.head.appendChild(meta);
    
    global.fetch = jest.fn(() =>
      Promise.resolve({
        json: () => Promise.resolve({ success: true }),
      })
    );
    
    await securePost('/api/data', { foo: 'bar' });
    
    expect(fetch).toHaveBeenCalledWith('/api/data', expect.objectContaining({
      headers: expect.objectContaining({
        'X-CSRF-Token': 'test-csrf-token-123',
      }),
    }));
  });
  
  test('throws error when CSRF token is missing', async () => {
    await expect(securePost('/api/data', {})).rejects.toThrow('CSRF token not found');
  });
});
```

**Example 5: Testing sensitive data handling**

```javascript
// CreditCardForm.jsx
export function CreditCardForm({ onSubmit }) {
  const [cardNumber, setCardNumber] = useState('');
  
  const maskCardNumber = (number) => {
    // Only show last 4 digits
    if (number.length <= 4) return number;
    return '****' + number.slice(-4);
  };
  
  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      onSubmit(cardNumber);
    }}>
      <input
        type="text"
        value={maskCardNumber(cardNumber)}
        onChange={(e) => {
          // Store full number internally but display masked
          const input = e.target.value.replace(/\*/g, '');
          setCardNumber(cardNumber.slice(0, -4) + input.slice(-4));
        }}
        placeholder="Card number"
        autoComplete="cc-number"
        inputMode="numeric"
      />
      <button type="submit">Submit</button>
    </form>
  );
}

// CreditCardForm.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { CreditCardForm } from './CreditCardForm';

describe('CreditCardForm Security', () => {
  test('masks card number in UI', async () => {
    const user = userEvent.setup();
    render(<CreditCardForm onSubmit={jest.fn()} />);
    
    const input = screen.getByPlaceholderText('Card number');
    
    await user.type(input, '1234567812345678');
    
    // Should show only last 4 digits
    expect(input.value).toContain('5678');
    expect(input.value).not.toContain('1234567812345678');
  });
  
  test('uses autocomplete="cc-number" for security', () => {
    render(<CreditCardForm onSubmit={jest.fn()} />);
    
    const input = screen.getByPlaceholderText('Card number');
    expect(input).toHaveAttribute('autocomplete', 'cc-number');
  });
  
  test('does not log sensitive data in console', () => {
    const consoleSpy = jest.spyOn(console, 'log');
    const onSubmit = jest.fn();
    
    render(<CreditCardForm onSubmit={onSubmit} />);
    
    // Verify no card numbers are logged
    expect(consoleSpy).not.toHaveBeenCalledWith(expect.stringContaining('1234567812345678'));
    
    consoleSpy.mockRestore();
  });
});
```

**Example 6: E2E security testing with Playwright**

```javascript
// e2e/security.spec.js
import { test, expect } from '@playwright/test';

test.describe('Security Tests', () => {
  test('has Content Security Policy headers', async ({ page }) => {
    const response = await page.goto('/');
    const csp = response.headers()['content-security-policy'];
    
    expect(csp).toBeDefined();
    expect(csp).toContain("default-src 'self'");
  });
  
  test('sets secure cookie flags', async ({ page, context }) => {
    await page.goto('/login');
    
    await page.fill('[name="username"]', 'testuser');
    await page.fill('[name="password"]', 'password123');
    await page.click('button[type="submit"]');
    
    const cookies = await context.cookies();
    const sessionCookie = cookies.find(c => c.name === 'session');
    
    expect(sessionCookie).toBeDefined();
    expect(sessionCookie.secure).toBe(true);
    expect(sessionCookie.httpOnly).toBe(true);
    expect(sessionCookie.sameSite).toBe('Strict');
  });
  
  test('prevents clickjacking with X-Frame-Options', async ({ page }) => {
    const response = await page.goto('/');
    const xFrameOptions = response.headers()['x-frame-options'];
    
    expect(xFrameOptions).toBe('DENY');
  });
  
  test('requires HTTPS', async ({ page }) => {
    const response = await page.goto('/');
    const strictTransportSecurity = response.headers()['strict-transport-security'];
    
    expect(strictTransportSecurity).toBeDefined();
    expect(strictTransportSecurity).toContain('max-age=');
  });
  
  test('sanitizes user input before display', async ({ page }) => {
    await page.goto('/comments');
    
    const maliciousInput = '<img src=x onerror=alert("XSS")>';
    await page.fill('[name="comment"]', maliciousInput);
    await page.click('button[type="submit"]');
    
    // Check that script did not execute
    const alerts = [];
    page.on('dialog', dialog => alerts.push(dialog.message()));
    
    await page.waitForTimeout(1000);
    expect(alerts).toHaveLength(0);
    
    // Verify content is escaped
    const content = await page.textContent('.comment');
    expect(content).not.toContain('<img');
  });
});
```

**Best Practices:**
- ✅ Sanitize all user input
- ✅ Use DOMPurify for HTML sanitization
- ✅ Validate on both client and server
- ✅ Test authentication and authorization flows
- ✅ Verify CSRF tokens are required
- ✅ Test with malicious inputs (XSS, SQL injection patterns)
- ✅ Check secure headers (CSP, X-Frame-Options, HSTS)
- ✅ Never expose sensitive data in DOM/logs
- ✅ Run `npm audit` regularly
- ❌ Don't trust client-side validation alone
- ❌ Don't log sensitive information
- ❌ Don't render raw HTML without sanitization

---

## Testing Tools Ecosystem

A comprehensive overview of testing tools and when to use each.

### Test Runners

| Tool | Best For | Key Features |
|------|----------|--------------|
| **Jest** | General React testing | Zero config, snapshot testing, mocking, parallel tests, coverage |
| **Vitest** | Vite projects | Fast, ESM support, Jest-compatible API, HMR for tests |
| **Mocha** | Flexibility | Pluggable, works with any assertion library, minimal |

**Recommendation:** Jest for most React projects, Vitest if using Vite.

### Component Testing Libraries

| Tool | Best For | Key Features |
|------|----------|--------------|
| **React Testing Library** | User-centric testing | Query by accessible roles, encourages best practices, no implementation details |
| **Enzyme** | Legacy projects | Shallow rendering, direct state access (discouraged now) |
| **@testing-library/user-event** | Realistic interactions | Simulates real user events (keyboard, mouse, clipboard) |

**Recommendation:** React Testing Library + @testing-library/user-event.

### E2E Testing Frameworks

| Tool | Best For | Key Features |
|------|----------|--------------|
| **Playwright** | Modern E2E testing | Multi-browser, auto-wait, powerful selectors, video/screenshots, parallelization |
| **Cypress** | Developer experience | Time-travel debugging, automatic waiting, real-time reloads, network stubbing |
| **Selenium** | Cross-browser legacy | Mature, multi-language support, wide browser support |
| **Puppeteer** | Chrome-only automation | Headless Chrome, fast, good for scraping/PDF generation |

**Recommendation:** Playwright for new projects (best performance/features), Cypress for great DX.

### API/Network Mocking

| Tool | Best For | Key Features |
|------|----------|--------------|
| **MSW (Mock Service Worker)** | Network-level mocking | Service worker, works in browser + Node, realistic, framework-agnostic |
| **nock** | Node.js HTTP mocking | HTTP request interception, recording/playback |
| **jest.mock()** | Simple mocks | Built into Jest, good for basic mocking |
| **miragejs** | Full API simulation | In-memory database, relationships, factories |

**Recommendation:** MSW for most API testing (most realistic).

### Snapshot Testing

| Tool | Best For | Key Features |
|------|----------|--------------|
| **Jest Snapshots** | Component structure | Built-in, easy to use, inline snapshots |
| **Storybook** | Component documentation | Visual documentation, isolation, addons |
| **jest-snapshot-serializer** | Custom serialization | Clean snapshots, remove noise |

**Recommendation:** Jest snapshots for simple cases, Storybook for design systems.

### Visual Regression Testing

| Tool | Best For | Key Features |
|------|----------|--------------|
| **Playwright** | Built-in screenshots | Fast, reliable, multi-browser, threshold tolerance |
| **Chromatic** | Storybook integration | CI/CD integration, UI review workflow, component library |
| **Percy** | Cross-browser visuals | Responsive testing, visual diffs, integrations |
| **BackstopJS** | Open-source solution | Headless browser, free, configurable scenarios |
| **Applitools** | AI-powered testing | Smart diffs, cross-browser, layout testing |

**Recommendation:** Playwright for in-house, Chromatic/Percy for design systems.

### Accessibility Testing

| Tool | Best For | Key Features |
|------|----------|--------------|
| **jest-axe** | Unit/component tests | Automated a11y checks, integrates with Jest |
| **@axe-core/playwright** | E2E a11y testing | Full-page scans, detailed reports |
| **eslint-plugin-jsx-a11y** | Static analysis | Catch issues during development, IDE integration |
| **pa11y** | CI/CD integration | CLI tool, automated audits |
| **WAVE** | Manual testing | Browser extension, visual feedback |

**Recommendation:** jest-axe + eslint-plugin-jsx-a11y + manual testing.

### Performance Testing

| Tool | Best For | Key Features |
|------|----------|--------------|
| **Lighthouse CI** | Web vitals auditing | Core Web Vitals, performance budgets, CI integration |
| **size-limit** | Bundle size monitoring | Pre-commit checks, size tracking, time to execute |
| **bundlesize** | Bundle size limits | CI integration, compression simulation |
| **React DevTools Profiler** | Component performance | Render timing, flame graphs, commit analysis |
| **web-vitals** | Real user metrics | LCP, FID, CLS measurement |

**Recommendation:** Lighthouse CI + size-limit for comprehensive monitoring.

### State Management Testing

| Tool | Best For | Key Features |
|------|----------|--------------|
| **@reduxjs/toolkit** | Redux testing | Built-in testing utilities, mock store |
| **zustand** | Simple store testing | Direct testing, no provider needed |
| **React Testing Library** | Context testing | Wrap with providers, test behavior |

**Recommendation:** Test stores in isolation, then integration with components.

### Code Coverage

| Tool | Best For | Key Features |
|------|----------|--------------|
| **Jest Coverage** | Built-in coverage | Statement/branch/function/line coverage, HTML reports |
| **c8** | Native V8 coverage | Faster, accurate, works with ESM |
| **Codecov/Coveralls** | CI/CD reporting | PR comments, trend tracking, team dashboards |

**Recommendation:** Jest coverage locally, Codecov/Coveralls for teams.

### Testing Utilities

| Tool | Best For | Key Features |
|------|----------|--------------|
| **faker.js / @faker-js/faker** | Test data generation | Realistic fake data, localization |
| **test-data-bot** | Factory pattern | Type-safe, customizable, composable |
| **chance.js** | Random data | Numbers, strings, dates, names |
| **factory-bot** | Rails-style factories | Relationships, sequences, traits |

**Recommendation:** @faker-js/faker for most test data needs.

### Debugging Tools

| Tool | Best For | Key Features |
|------|----------|--------------|
| **React DevTools** | Component inspection | Props/state inspection, profiler, hooks |
| **Testing Playground** | Query debugging | Find best queries, accessibility tree |
| **@testing-library/dom debug()** | DOM output | Pretty-print DOM, syntax highlighting |
| **Playwright Inspector** | E2E debugging | Step through tests, pick selectors, time-travel |

**Recommendation:** Use all - each serves a different debugging need.

### CI/CD Integration

| Tool | Best For | Key Features |
|------|----------|--------------|
| **GitHub Actions** | GitHub projects | Native integration, free minutes, marketplace |
| **CircleCI** | Docker workflows | Fast, caching, parallelization, orbs |
| **GitLab CI** | GitLab projects | Built-in, free runners, auto DevOps |
| **Jenkins** | Self-hosted | Highly customizable, plugins, on-premise |

**Recommendation:** GitHub Actions for simplicity, CircleCI for speed.

### Recommended Stack by Project Type

**Small Projects / Prototypes:**
```
Jest + React Testing Library
```

**Medium Projects:**
```
Jest + React Testing Library + @testing-library/user-event
+ MSW (API mocking)
+ Playwright (E2E for critical paths)
```

**Large Projects / Design Systems:**
```
Jest + React Testing Library + @testing-library/user-event
+ MSW (API mocking)
+ Playwright (comprehensive E2E)
+ Chromatic (visual regression)
+ jest-axe (accessibility)
+ Lighthouse CI (performance)
+ size-limit (bundle monitoring)
+ GitHub Actions (CI/CD)
```

**Enterprise / Mission-Critical:**
```
All of the above, plus:
+ Percy or Applitools (cross-browser visual testing)
+ Codecov (coverage tracking)
+ OWASP ZAP (security scanning)
+ Datadog/Sentry (real-user monitoring)
+ Storybook (component documentation)
```

---

## Examples

---

## Common pitfalls

### 1. Testing Implementation Details

**Problem:** Testing internal state, private methods, or component structure instead of user behavior.

**Why it's bad:** Tests break when refactoring, even if functionality is unchanged.

**Example (Bad):**
```javascript
// ❌ Testing implementation details
test('increments counter state', () => {
  const { result } = renderHook(() => useCounter());
  
  act(() => result.current.increment());
  
  // Testing internal state
  expect(result.current.count).toBe(1);
});
```

**Better approach:**
```javascript
// ✅ Testing user behavior
test('displays incremented count', () => {
  render(<Counter />);
  
  userEvent.click(screen.getByRole('button', { name: 'Increment' }));
  
  expect(screen.getByText('Count: 1')).toBeInTheDocument();
});
```

---

### 2. Over-Mocking

**Problem:** Mocking too much makes tests less valuable and more brittle.

**Why it's bad:** You're testing mocks, not real code behavior.

**Example (Bad):**
```javascript
// ❌ Over-mocking
jest.mock('./UserService');
jest.mock('./AuthService');
jest.mock('./AnalyticsService');
jest.mock('./LoggingService');

// Now you're just testing mock implementations
```

**Better approach:**
```javascript
// ✅ Mock only external dependencies
// Use MSW for API calls, test real component logic
server.use(
  http.get('/api/user', () => HttpResponse.json({ name: 'Alice' }))
);

render(<UserProfile />);
```

---

### 3. Not Testing Edge Cases

**Problem:** Only testing the happy path.

**Why it's bad:** Bugs often occur in edge cases.

**Common missed scenarios:**
- Empty arrays/objects
- Null/undefined values
- Very long strings
- Special characters
- Network errors
- Loading states
- Permission denied
- Concurrent operations

**Example:**
```javascript
// ✅ Test edge cases
describe('UserList', () => {
  test('shows empty state when no users', () => {
    render(<UserList users={[]} />);
    expect(screen.getByText('No users found')).toBeInTheDocument();
  });
  
  test('handles null users prop', () => {
    render(<UserList users={null} />);
    expect(screen.getByText('No users found')).toBeInTheDocument();
  });
  
  test('handles very long names', () => {
    const longName = 'A'.repeat(1000);
    render(<UserList users={[{ id: 1, name: longName }]} />);
    // Verify it doesn't break layout
  });
});
```

---

### 4. Snapshot Overuse

**Problem:** Using snapshots for everything, accepting changes blindly.

**Why it's bad:** Snapshots become noise, hide real regressions.

**Bad practice:**
```javascript
// ❌ Snapshot entire component
test('renders correctly', () => {
  const { container } = render(<ComplexComponent />);
  expect(container).toMatchSnapshot();
});
```

**Better approach:**
```javascript
// ✅ Use targeted assertions
test('displays user information', () => {
  render(<UserCard user={{ name: 'Alice', email: 'alice@example.com' }} />);
  
  expect(screen.getByText('Alice')).toBeInTheDocument();
  expect(screen.getByText('alice@example.com')).toBeInTheDocument();
});

// ✅ Snapshot only specific data structures
test('formats user data correctly', () => {
  const result = formatUserData(rawData);
  expect(result).toMatchInlineSnapshot(`
    {
      "displayName": "Alice Smith",
      "role": "admin",
    }
  `);
});
```

---

### 5. Ignoring Test Performance

**Problem:** Tests take too long to run, slowing down development.

**Common causes:**
- Real timers instead of fake timers
- Unnecessary delays
- Not cleaning up after tests
- Sequential instead of parallel tests

**Solutions:**
```javascript
// ✅ Use fake timers
jest.useFakeTimers();

test('debounces input', () => {
  render(<SearchInput />);
  
  userEvent.type(screen.getByRole('textbox'), 'test');
  
  // Fast-forward time instead of waiting
  jest.advanceTimersByTime(500);
  
  expect(mockSearch).toHaveBeenCalledWith('test');
});

// ✅ Clean up after tests
afterEach(() => {
  jest.clearAllMocks();
  cleanup();
});

// ✅ Run tests in parallel (Jest does this by default)
// But avoid shared state between tests
```

---

### 6. Not Testing Accessibility

**Problem:** Skipping accessibility tests until later (or never).

**Why it's bad:** A11y bugs are harder to fix later, legal/ethical issues.

**Solution:**
```javascript
// ✅ Include a11y in every component test
test('is accessible', async () => {
  const { container } = render(<LoginForm />);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});

// ✅ Test keyboard navigation
test('can navigate with keyboard', async () => {
  render(<Menu />);
  
  await userEvent.tab(); // Focus first item
  expect(screen.getByRole('menuitem', { name: 'Home' })).toHaveFocus();
  
  await userEvent.keyboard('{ArrowDown}');
  expect(screen.getByRole('menuitem', { name: 'About' })).toHaveFocus();
});
```

---

### 7. Testing Third-Party Libraries

**Problem:** Writing tests for React Router, Redux, or other libraries.

**Why it's bad:** Waste of time, libraries are already tested.

**Example (Bad):**
```javascript
// ❌ Don't test that useState works
test('useState updates state', () => {
  // This tests React, not your code
});
```

**Better approach:**
```javascript
// ✅ Test YOUR logic that uses the library
test('navigates to dashboard after login', async () => {
  render(<LoginForm />);
  
  await userEvent.click(screen.getByRole('button', { name: 'Login' }));
  
  expect(screen.getByText('Dashboard')).toBeInTheDocument();
});
```

---

### 8. Flaky Tests

**Problem:** Tests that sometimes pass, sometimes fail.

**Common causes:**
- Race conditions
- Not waiting for async operations
- Tests depending on order
- Random data without seeds
- Timeouts too short
- Shared state between tests

**Solutions:**
```javascript
// ❌ Flaky - doesn't wait
test('loads user data', () => {
  render(<UserProfile />);
  expect(screen.getByText('Alice')).toBeInTheDocument(); // Might not be loaded yet
});

// ✅ Wait for async operations
test('loads user data', async () => {
  render(<UserProfile />);
  expect(await screen.findByText('Alice')).toBeInTheDocument();
});

// ✅ Use seeds for random data
const faker = require('@faker-js/faker');
faker.seed(123); // Reproducible random data

// ✅ Isolate tests
beforeEach(() => {
  // Reset everything
  jest.clearAllMocks();
  localStorage.clear();
  sessionStorage.clear();
});
```

---

### 9. Not Testing Error States

**Problem:** Only testing when everything works.

**Why it's bad:** Error handling is critical for UX.

**Example:**
```javascript
// ✅ Test error scenarios
test('displays error message when fetch fails', async () => {
  server.use(
    http.get('/api/users', () => {
      return new HttpResponse(null, { status: 500 });
    })
  );
  
  render(<UserList />);
  
  expect(await screen.findByRole('alert')).toHaveTextContent('Failed to load users');
});

test('allows retry after error', async () => {
  server.use(
    http.get('/api/users', () => {
      return new HttpResponse(null, { status: 500 });
    })
  );
  
  render(<UserList />);
  
  await screen.findByRole('alert');
  
  // Fix the API
  server.use(
    http.get('/api/users', () => {
      return HttpResponse.json([{ id: 1, name: 'Alice' }]);
    })
  );
  
  await userEvent.click(screen.getByRole('button', { name: 'Retry' }));
  
  expect(await screen.findByText('Alice')).toBeInTheDocument();
});
```

---

### 10. Writing Tests After Code

**Problem:** Not following TDD, writing tests as an afterthought.

**Why it's bad:** Tests become less effective, harder to write, lower coverage.

**Better approach:**
1. Write failing test first (Red)
2. Write minimal code to pass (Green)
3. Refactor (Refactor)
4. Repeat

---

## Quick self-check

Test your understanding with these questions:

### TDD Fundamentals

1. **What are the three steps of the Red-Green-Refactor cycle?**
   <details>
   <summary>Answer</summary>
   
   1. **Red**: Write a failing test
   2. **Green**: Write minimal code to make it pass
   3. **Refactor**: Improve code quality without changing behavior
   </details>

2. **Why write the test before the code?**
   <details>
   <summary>Answer</summary>
   
   - Ensures code is testable from the start
   - Clarifies requirements before implementation
   - Prevents over-engineering
   - Guarantees test actually tests the code (not a false positive)
   - Improves design through thinking about API first
   </details>

3. **When should you NOT use TDD?**
   <details>
   <summary>Answer</summary>
   
   - Spike/exploratory work
   - UI design/prototyping
   - Learning new technology
   - Performance-critical code requiring profiling
   - Very simple boilerplate code
   </details>

### Test Types

4. **What's the difference between a unit test and an integration test?**
   <details>
   <summary>Answer</summary>
   
   - **Unit test**: Tests a single function/component in isolation, mocks dependencies
   - **Integration test**: Tests multiple components/modules working together, uses real dependencies where possible
   </details>

5. **When would you use a snapshot test vs. a visual regression test?**
   <details>
   <summary>Answer</summary>
   
   - **Snapshot test**: Check data structures or simple HTML output, fast, text-based
   - **Visual regression test**: Check actual rendered pixels, catch visual bugs, slower, image-based
   </details>

6. **What are the Core Web Vitals you should test?**
   <details>
   <summary>Answer</summary>
   
   - **LCP (Largest Contentful Paint)**: < 2.5s
   - **FID (First Input Delay)**: < 100ms (or INP < 200ms)
   - **CLS (Cumulative Layout Shift)**: < 0.1
   </details>

### Testing Best Practices

7. **What's wrong with this test?**
   ```javascript
   test('counter works', () => {
     const { result } = renderHook(() => useCounter());
     expect(result.current.count).toBe(0);
     act(() => result.current.increment());
     expect(result.current.count).toBe(1);
   });
   ```
   <details>
   <summary>Answer</summary>
   
   It tests implementation details (internal state) instead of user behavior. Better to render the component and test what the user sees.
   </details>

8. **What's the Testing Trophy/Pyramid concept?**
   <details>
   <summary>Answer</summary>
   
   - **Pyramid (traditional)**: Many unit tests, some integration, few E2E
   - **Trophy (modern)**: Some unit, MANY integration, some E2E, few static
   - Focus on integration tests for best ROI
   </details>

9. **How do you test async code properly?**
   <details>
   <summary>Answer</summary>
   
   - Use `async/await` with `findBy*` queries
   - Use `waitFor` for complex assertions
   - Use MSW for network requests
   - Don't use arbitrary `setTimeout`
   - Clean up after tests with `cleanup()`
   </details>

### Tools & Ecosystem

10. **When would you choose Playwright over Cypress?**
    <details>
    <summary>Answer</summary>
    
    - **Playwright**: Multi-browser support, better parallelization, faster, better API, headless by default
    - **Cypress**: Better DX, time-travel debugging, easier for beginners, better documentation
    - Choose Playwright for new projects needing performance/multi-browser
    </details>

11. **What's MSW and why use it instead of jest.mock()?**
    <details>
    <summary>Answer</summary>
    
    - **MSW (Mock Service Worker)**: Intercepts network requests at the network layer using service workers
    - **Advantages**: Framework-agnostic, works in browser and Node, more realistic, reusable across test types, same handlers for development
    </details>

12. **What's the difference between jest-axe and manual accessibility testing?**
    <details>
    <summary>Answer</summary>
    
    - **jest-axe**: Catches ~30-40% of a11y issues automatically (missing alt text, color contrast, ARIA roles)
    - **Manual testing**: Required for keyboard nav, screen reader experience, focus management, logical tab order
    - Need BOTH for comprehensive a11y coverage
    </details>

### Security Testing

13. **Name 3 XSS attack vectors you should test for.**
    <details>
    <summary>Answer</summary>
    
    1. `<script>` tags in user input
    2. `javascript:` URLs
    3. Event handlers (`onclick`, `onerror`, etc.)
    4. Bonus: `<iframe>`, data URIs, SVG with embedded scripts
    </details>

14. **What security headers should you verify in E2E tests?**
    <details>
    <summary>Answer</summary>
    
    - Content-Security-Policy (CSP)
    - X-Frame-Options (clickjacking protection)
    - Strict-Transport-Security (HSTS)
    - X-Content-Type-Options
    - Secure cookie flags (Secure, HttpOnly, SameSite)
    </details>

---

## Further practice

### Exercises

**Beginner:**

1. **Write a TDD Calculator**
   - Create `add`, `subtract`, `multiply`, `divide` functions using TDD
   - Handle edge cases: division by zero, invalid inputs, very large numbers
   - Practice Red-Green-Refactor cycle

2. **Test a Todo List Component**
   - Add todos
   - Mark complete/incomplete
   - Delete todos
   - Filter (all/active/completed)
   - Test with React Testing Library

3. **Test Form Validation**
   - Email validation
   - Password strength
   - Confirm password match
   - Display error messages
   - Test with user-event

**Intermediate:**

4. **Test an Authenticated Flow**
   - Login form
   - Protected routes
   - Logout
   - Session expiration
   - Use MSW for API mocking

5. **Visual Regression Testing**
   - Setup Playwright or Chromatic
   - Test a component library (Button, Card, Modal)
   - Test responsive layouts
   - Test dark/light themes

6. **Performance Testing**
   - Test a virtualized list with 10,000 items
   - Monitor bundle size with size-limit
   - Setup Lighthouse CI
   - Measure Core Web Vitals

**Advanced:**

7. **Test Complex State Management**
   - Shopping cart with Redux/Zustand
   - Optimistic updates
   - Undo/redo functionality
   - Persistence to localStorage
   - Test reducers, selectors, and UI together

8. **E2E Testing Suite**
   - Full checkout flow (browse → add to cart → checkout → payment → confirmation)
   - Test with Playwright
   - Handle authentication
   - Mock payment API
   - Generate test report

9. **Accessibility Audit**
   - Run jest-axe on all components
   - Test keyboard navigation throughout app
   - Test with screen reader (NVDA/JAWS)
   - Fix all WCAG AA violations
   - Document a11y patterns

10. **Security Testing**
    - Test XSS prevention
    - Test CSRF protection
    - Test input sanitization
    - Verify security headers
    - Run OWASP ZAP scan

### Katas (Practice repeatedly)

1. **String Calculator (TDD Classic)**
   - Input: "1,2,3" → Output: 6
   - Handle new lines: "1\n2,3" → 6
   - Custom delimiter: "//;\n1;2" → 3
   - Negative numbers throw exception

2. **FizzBuzz (Component Testing)**
   - Display numbers 1-100
   - Multiples of 3: "Fizz"
   - Multiples of 5: "Buzz"
   - Multiples of both: "FizzBuzz"
   - Test rendering and logic

3. **Roman Numeral Converter (TDD)**
   - 1 → I, 4 → IV, 9 → IX
   - 27 → XXVII, 48 → XLVIII
   - Bidirectional conversion

### Real-World Scenarios

**Scenario 1: Flaky Tests**
- You have a test suite where 5-10% of tests fail randomly
- Tasks:
  - Identify common causes
  - Fix race conditions
  - Add proper waits
  - Ensure test isolation

**Scenario 2: Legacy Codebase**
- 0% test coverage, tightly coupled code
- Tasks:
  - Start with E2E tests for critical paths
  - Add integration tests as you refactor
  - Gradually increase unit test coverage
  - Use characterization tests

**Scenario 3: Slow Test Suite**
- 500 tests taking 20 minutes
- Tasks:
  - Profile tests to find slow ones
  - Use fake timers
  - Parallelize where possible
  - Mock expensive operations
  - Consider splitting test suites

### Learning Resources

**Books:**
- *Test Driven Development: By Example* - Kent Beck
- *Growing Object-Oriented Software, Guided by Tests* - Freeman & Pryce
- *The Art of Unit Testing* - Roy Osherove

**Articles/Guides:**
- [Testing Library Documentation](https://testing-library.com/)
- [Kent C. Dodds - Common Testing Mistakes](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library)
- [Martin Fowler - Test Pyramid](https://martinfowler.com/bliki/TestPyramid.html)

**Video Courses:**
- [Testing JavaScript (Kent C. Dodds)](https://testingjavascript.com/)
- [Epic React Testing Module](https://epicreact.dev/)

**Practice Platforms:**
- [Exercism.io](https://exercism.io/) - TDD practice
- [Codewars](https://www.codewars.com/) - Kata practice
- [Testing Playground](https://testing-playground.com/) - Query practice

---

## Interview preparation

### Common Interview Questions

**Conceptual Questions:**

1. **"What is Test-Driven Development (TDD)?"**
   
   **Strong answer:** TDD is a development practice where you write a failing test before writing production code. The cycle is Red-Green-Refactor: write a failing test (Red), write minimal code to pass it (Green), then improve the code quality (Refactor). Benefits include better design, higher confidence, living documentation, and fewer bugs.

2. **"What's the difference between unit, integration, and E2E tests?"**
   
   **Strong answer:**
   - **Unit tests**: Test individual functions/components in isolation with mocked dependencies. Fast, focused, but can miss integration issues.
   - **Integration tests**: Test multiple components working together with real or realistic dependencies. Slower but higher confidence.
   - **E2E tests**: Test entire user flows in a real browser. Slowest but catch real-world issues. Follow the Testing Trophy—focus on integration tests for best ROI.

3. **"How do you test asynchronous code in React?"**
   
   **Strong answer:** Use `findBy*` queries that return promises and wait for elements to appear. For complex scenarios, use `waitFor`. Always use `async/await`. Mock network requests with MSW at the network level rather than mocking fetch directly. Avoid arbitrary timeouts—use fake timers if you need to control time.
   
   ```javascript
   test('loads user data', async () => {
     render(<UserProfile />);
     expect(await screen.findByText('Alice')).toBeInTheDocument();
   });
   ```

4. **"What are testing anti-patterns?"**
   
   **Strong answer:**
   - Testing implementation details (internal state, private methods)
   - Over-mocking (testing mocks instead of real code)
   - Flaky tests (inconsistent results)
   - Snapshot overuse
   - Not testing edge cases or error states
   - Tests depending on execution order
   - Writing tests after code

5. **"How do you test accessibility?"**
   
   **Strong answer:** Multi-layered approach:
   - Static analysis with `eslint-plugin-jsx-a11y`
   - Automated tests with `jest-axe` for WCAG violations
   - Keyboard navigation testing with `user-event`
   - Manual screen reader testing
   - Check ARIA attributes, focus management, semantic HTML
   - Automated tools catch ~30-40%, manual testing is essential

**Practical Coding Questions:**

6. **"Write a test for a login form that shows an error on invalid credentials."**
   
   ```javascript
   test('displays error on invalid credentials', async () => {
     server.use(
       http.post('/api/login', () => {
         return new HttpResponse(null, { status: 401 });
       })
     );
     
     render(<LoginForm />);
     
     await userEvent.type(screen.getByLabelText('Email'), 'wrong@example.com');
     await userEvent.type(screen.getByLabelText('Password'), 'wrongpass');
     await userEvent.click(screen.getByRole('button', { name: 'Login' }));
     
     expect(await screen.findByRole('alert')).toHaveTextContent('Invalid credentials');
   });
   ```

7. **"How would you test a component that fetches data on mount?"**
   
   ```javascript
   test('fetches and displays user data on mount', async () => {
     server.use(
       http.get('/api/user/123', () => {
         return HttpResponse.json({ id: 123, name: 'Alice' });
       })
     );
     
     render(<UserProfile userId="123" />);
     
     // Show loading state
     expect(screen.getByText('Loading...')).toBeInTheDocument();
     
     // Show loaded data
     expect(await screen.findByText('Alice')).toBeInTheDocument();
     expect(screen.queryByText('Loading...')).not.toBeInTheDocument();
   });
   ```

8. **"Write a test to verify a button is disabled during loading."**
   
   ```javascript
   test('disables submit button during form submission', async () => {
     render(<CreateUserForm />);
     
     const submitButton = screen.getByRole('button', { name: 'Create User' });
     
     await userEvent.type(screen.getByLabelText('Name'), 'Alice');
     await userEvent.click(submitButton);
     
     // Button should be disabled while submitting
     expect(submitButton).toBeDisabled();
     
     // Wait for submission to complete
     await waitFor(() => expect(submitButton).not.toBeDisabled());
   });
   ```

**Behavioral Questions:**

9. **"Tell me about a time when tests caught a critical bug."**
   
   **STAR format:**
   - **Situation:** "In my previous project, we had a payment processing form..."
   - **Task:** "I needed to ensure edge cases were covered..."
   - **Action:** "I wrote integration tests for error scenarios including network failures, invalid payment methods, and timeout conditions..."
   - **Result:** "During staging deployment, tests caught a race condition where double-clicking submit would charge twice. This would have been a serious production issue affecting revenue and customer trust."

10. **"How do you convince a team to adopt TDD?"**
    
    **Good answer:**
    - Start small with a pilot project or feature
    - Show concrete benefits: fewer bugs found in QA/production, faster debugging, better design
    - Pair programming to teach TDD
    - Measure metrics: reduced bug count, faster PR reviews, improved velocity over time
    - Address concerns: yes, it's slower initially, but pays off quickly
    - Lead by example—write great tests and let the team see the value

**Advanced Questions:**

11. **"How do you test React hooks?"**
    
    **Strong answer:** "I prefer testing hooks through components that use them, since that's how they're actually used. But for complex reusable hooks, I use `@testing-library/react-hooks` with `renderHook`. Always wrap state updates in `act()` and test the hook's public API, not implementation."
    
    ```javascript
    test('useDebounce delays value update', () => {
      jest.useFakeTimers();
      const { result, rerender } = renderHook(
        ({ value }) => useDebounce(value, 500),
        { initialProps: { value: 'initial' } }
      );
      
      expect(result.current).toBe('initial');
      
      rerender({ value: 'updated' });
      expect(result.current).toBe('initial'); // Still old value
      
      jest.advanceTimersByTime(500);
      expect(result.current).toBe('updated'); // Now updated
    });
    ```

12. **"How do you handle test data management in a large test suite?"**
    
    **Strong answer:**
    - Use factories (test-data-bot) or @faker-js/faker for consistent test data
    - Seed random generators for reproducibility
    - Create builder patterns for complex objects
    - Share fixtures but avoid global state
    - Reset databases/stores between tests
    - Use meaningful data that makes tests readable

13. **"What's your strategy for testing a legacy codebase with no tests?"**
    
    **Strong answer:**
    1. Start with E2E tests for critical user paths (high value, low effort)
    2. Add characterization tests when touching legacy code
    3. Write integration tests as you refactor
    4. Gradually increase unit test coverage for new/changed code
    5. Don't aim for 100%—focus on business-critical and complex code
    6. Use coverage tools to find gaps, but don't worship the number

### What Interviewers Look For

✅ **Understanding of testing philosophy:**
- Why test (confidence, documentation, design)
- What to test (behavior over implementation)
- How much to test (pragmatic, ROI-focused)

✅ **Practical experience:**
- Can write clean, readable tests
- Knows when to mock vs. use real dependencies
- Handles async code properly
- Tests accessibility and edge cases

✅ **Tool knowledge:**
- React Testing Library best practices
- Jest/Vitest configuration
- E2E framework (Playwright/Cypress)
- Mocking strategies (MSW)

✅ **Problem-solving:**
- Debugs flaky tests
- Improves slow test suites
- Balances speed vs. confidence
- Adapts testing strategy to project needs

### Red Flags to Avoid

❌ "Tests are a waste of time"
❌ "I just test manually"
❌ "100% coverage is the goal"
❌ "Integration tests are too slow"
❌ Testing implementation details
❌ No knowledge of testing tools/frameworks
❌ Can't explain TDD cycle
❌ Never written E2E tests

### Quick Tips for Success

1. **Practice live coding tests** - time yourself, write tests on a whiteboard
2. **Know your tools** - RTL queries, waitFor, userEvent, MSW setup
3. **Think user-first** - what would a user see/do?
4. **Communicate your thinking** - explain why you're choosing a specific test type
5. **Ask clarifying questions** - "Should I test error cases?" "Do you want E2E or unit tests?"
6. **Show pragmatism** - balance perfect tests with shipping value

**Sample interview preparation checklist:**
- [ ] Can write a unit test for a function in 5 minutes
- [ ] Can test a form with validation
- [ ] Can test async API calls with MSW
- [ ] Can explain TDD cycle clearly
- [ ] Know when to use unit vs integration vs E2E
- [ ] Can test accessibility with jest-axe
- [ ] Can debug a flaky test
- [ ] Understand mocking strategies
- [ ] Can set up basic E2E test with Playwright
- [ ] Know performance testing basics (Lighthouse, bundle size)

---

**Remember:** In interviews, demonstrating a balanced, pragmatic approach to testing is more valuable than claiming to test everything perfectly. Show you understand trade-offs and make decisions based on business value and risk.