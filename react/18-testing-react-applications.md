# Testing React Applications

**Level:** 🔥 Advanced  
**Tags:** `testing`, `jest`, `react-testing-library`, `unit-tests`, `integration-tests`, `interview`

---

## Why this matters

Testing is a critical skill that separates junior developers from senior engineers. Interviewers use testing questions to assess your understanding of component behavior, best practices, and ability to write maintainable code. Companies that value quality expect candidates to write tests, and many make it part of the technical interview.

**Why interviewers care:**
- Testing questions reveal whether you write production-quality code vs. "works on my machine" code
- Test-writing ability shows you understand component contracts, edge cases, and user behavior
- Testing patterns (what to test, what not to test) separate experienced engineers from beginners
- Knowledge of testing tools (Jest, RTL) is expected at mid-level and above
- TDD/BDD discussions assess software engineering maturity
- Mocking and isolation strategies reveal architectural understanding

**Real-world implications:**
- **Confidence to refactor:** Tests catch regressions when you change code
- **Living documentation:** Tests describe how components should behave
- **Faster debugging:** Tests pinpoint exactly what broke
- **Code quality:** Testable code is usually well-designed code
- **Team collaboration:** Tests communicate component contracts to team members
- **CI/CD pipelines:** Automated tests prevent bad code from reaching production
- **Cost savings:** Catching bugs in tests is 10-100x cheaper than in production

**Common testing challenges:**
- What to test (implementation vs. behavior)
- Mocking API calls and external dependencies
- Testing async behavior
- Testing hooks
- Testing user interactions
- Achieving good coverage without slowing down development
- Brittle tests that break on every refactor

**What you must know:**
- **Jest:** JavaScript testing framework (test runner, assertions, mocks)
- **React Testing Library (RTL):** Recommended by React team, focuses on user behavior
- **Testing philosophy:** Test behavior, not implementation details
- **Queries:** getBy, queryBy, findBy and their variants
- **User events:** fireEvent vs. userEvent
- **Async testing:** waitFor, findBy queries
- **Mocking:** Mock modules, API calls, timers

**Interview red flags:** Testing implementation details (class names, component internals), not knowing the difference between unit and integration tests, or never having written tests.

---

## Core Ideas

### 1. **Testing Philosophy: Behavior over Implementation**

**❌ Bad: Testing implementation details**

```jsx
// Component
function Counter() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <p data-testid="count">{count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}

// BAD TEST: Tests implementation (state variable name)
test('counter starts at 0', () => {
  const { container } = render(<Counter />);
  const component = container.firstChild;
  expect(component.state.count).toBe(0);  // Tests internal state!
});
```

**Why it's bad:**
- Breaks when you refactor (e.g., rename `count` to `value`)
- Doesn't test what users actually see
- Couples tests to implementation

**✅ Good: Testing user-visible behavior**

```jsx
test('counter starts at 0', () => {
  render(<Counter />);
  expect(screen.getByText('0')).toBeInTheDocument();
});

test('clicking increment button increases count', () => {
  render(<Counter />);
  const button = screen.getByRole('button', { name: /increment/i });
  
  fireEvent.click(button);
  
  expect(screen.getByText('1')).toBeInTheDocument();
});
```

**Why it's good:**
- Tests what users see and do
- Survives refactoring (internal variable names can change)
- Tests the actual contract of the component

**Key principle:** *Your tests should resemble how users interact with your app.*

---

### 2. **React Testing Library: Queries**

RTL provides queries to find elements. They differ in behavior when elements aren't found.

**Query types:**

| Query Type | Returns | Async | Throws on no match |
|------------|---------|-------|-------------------|
| `getBy*` | Element | No | Yes ✅ |
| `queryBy*` | Element or null | No | No |
| `findBy*` | Promise<Element> | Yes ✅ | Yes ✅ |
| `getAllBy*` | Element[] | No | Yes ✅ |
| `queryAllBy*` | Element[] | No | No |
| `findAllBy*` | Promise<Element[]> | Yes ✅ | Yes ✅ |

**When to use each:**

```jsx
// getBy* - Element is there (throws if not)
const button = screen.getByRole('button', { name: /submit/i });
expect(button).toBeInTheDocument();

// queryBy* - Check element is NOT there (doesn't throw)
const error = screen.queryByText(/error/i);
expect(error).not.toBeInTheDocument();

// findBy* - Wait for async element to appear
const message = await screen.findByText(/success/i);
expect(message).toBeInTheDocument();
```

**Query priority (use in this order):**

1. **getByRole** - Preferred (accessibility-focused)
   ```jsx
   screen.getByRole('button', { name: /submit/i })
   screen.getByRole('textbox', { name: /email/i })
   screen.getByRole('heading', { level: 1 })
   ```

2. **getByLabelText** - For form fields
   ```jsx
   screen.getByLabelText(/email address/i)
   ```

3. **getByPlaceholderText** - Placeholder text
   ```jsx
   screen.getByPlaceholderText(/enter email/i)
   ```

4. **getByText** - Text content
   ```jsx
   screen.getByText(/welcome back/i)
   ```

5. **getByTestId** - Last resort (not user-visible)
   ```jsx
   screen.getByTestId('custom-element')
   ```

**Example:**

```jsx
function LoginForm() {
  return (
    <form>
      <label htmlFor="email">Email</label>
      <input id="email" type="email" placeholder="you@example.com" />
      
      <button type="submit">Login</button>
    </form>
  );
}

test('login form renders correctly', () => {
  render(<LoginForm />);
  
  // Preferred: getByRole (accessibility)
  const button = screen.getByRole('button', { name: /login/i });
  expect(button).toBeInTheDocument();
  
  // Good: getByLabelText (associated label)
  const emailInput = screen.getByLabelText(/email/i);
  expect(emailInput).toBeInTheDocument();
  
  // Okay: getByPlaceholderText (visible to users)
  const input = screen.getByPlaceholderText(/you@example.com/i);
  expect(input).toBeInTheDocument();
});
```

---

### 3. **Testing User Interactions**

**fireEvent vs. userEvent:**

```jsx
import { render, screen, fireEvent } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

// fireEvent - Low-level DOM events (legacy)
fireEvent.click(button);
fireEvent.change(input, { target: { value: 'test' } });

// userEvent - High-level user interactions (PREFERRED)
const user = userEvent.setup();
await user.click(button);
await user.type(input, 'test');
```

**Why userEvent is better:**
- Simulates real user behavior (click → focus → mousedown → mouseup → click)
- Fires all associated events in correct order
- More realistic testing

**Example: Testing a form**

```jsx
function ContactForm({ onSubmit }) {
  const [email, setEmail] = useState('');
  const [message, setMessage] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    onSubmit({ email, message });
  };

  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="email">Email</label>
      <input
        id="email"
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />

      <label htmlFor="message">Message</label>
      <textarea
        id="message"
        value={message}
        onChange={(e) => setMessage(e.target.value)}
      />

      <button type="submit">Send</button>
    </form>
  );
}

// TEST
test('submits form with email and message', async () => {
  const handleSubmit = jest.fn();
  render(<ContactForm onSubmit={handleSubmit} />);
  const user = userEvent.setup();

  // Type into inputs
  await user.type(screen.getByLabelText(/email/i), 'test@example.com');
  await user.type(screen.getByLabelText(/message/i), 'Hello world');

  // Submit form
  await user.click(screen.getByRole('button', { name: /send/i }));

  // Assert onSubmit was called with correct data
  expect(handleSubmit).toHaveBeenCalledWith({
    email: 'test@example.com',
    message: 'Hello world'
  });
});
```

---

### 4. **Testing Async Behavior**

**Problem:** Component fetches data after mounting.

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        setUser(data);
        setLoading(false);
      });
  }, [userId]);

  if (loading) return <div>Loading...</div>;
  
  return <div>{user.name}</div>;
}
```

**Solution: waitFor and findBy**

```jsx
import { render, screen, waitFor } from '@testing-library/react';

// Mock fetch
global.fetch = jest.fn(() =>
  Promise.resolve({
    json: () => Promise.resolve({ name: 'John Doe' })
  })
);

test('displays user name after loading', async () => {
  render(<UserProfile userId="123" />);

  // Initially shows loading
  expect(screen.getByText(/loading/i)).toBeInTheDocument();

  // Wait for user name to appear
  const userName = await screen.findByText('John Doe');
  expect(userName).toBeInTheDocument();

  // Loading should be gone
  expect(screen.queryByText(/loading/i)).not.toBeInTheDocument();
});

// Alternative: waitFor
test('displays user name after loading', async () => {
  render(<UserProfile userId="123" />);

  await waitFor(() => {
    expect(screen.getByText('John Doe')).toBeInTheDocument();
  });
});
```

**findBy vs. waitFor:**
- `findBy*` - Preferred for simple cases (waits for element to appear)
- `waitFor` - Use when you need custom logic or multiple assertions

---

### 5. **Mocking**

**Mock modules:**

```jsx
// services/api.js
export async function fetchUser(id) {
  const res = await fetch(`/api/users/${id}`);
  return res.json();
}

// UserProfile.jsx
import { fetchUser } from './services/api';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);

  return user ? <div>{user.name}</div> : <div>Loading...</div>;
}

// UserProfile.test.jsx
import { fetchUser } from './services/api';

jest.mock('./services/api');  // Mock the module

test('displays user name', async () => {
  // Mock the return value
  fetchUser.mockResolvedValue({ name: 'Jane Doe' });

  render(<UserProfile userId="456" />);

  const userName = await screen.findByText('Jane Doe');
  expect(userName).toBeInTheDocument();

  // Assert it was called with correct ID
  expect(fetchUser).toHaveBeenCalledWith('456');
});
```

**Mock timers:**

```jsx
function DelayedMessage() {
  const [show, setShow] = useState(false);

  useEffect(() => {
    const timer = setTimeout(() => setShow(true), 3000);
    return () => clearTimeout(timer);
  }, []);

  return show ? <div>Message appeared!</div> : null;
}

test('shows message after 3 seconds', () => {
  jest.useFakeTimers();
  
  render(<DelayedMessage />);
  
  // Message not there yet
  expect(screen.queryByText(/message appeared/i)).not.toBeInTheDocument();
  
  // Fast-forward 3 seconds
  jest.advanceTimersByTime(3000);
  
  // Message should appear
  expect(screen.getByText(/message appeared/i)).toBeInTheDocument();
  
  jest.useRealTimers();
});
```

---

### 6. **Testing Custom Hooks**

**Use @testing-library/react-hooks (or renderHook from RTL 13+)**

```jsx
// useCounter.js
import { useState } from 'react';

export function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);
  
  const increment = () => setCount(c => c + 1);
  const decrement = () => setCount(c => c - 1);
  const reset = () => setCount(initialValue);
  
  return { count, increment, decrement, reset };
}

// useCounter.test.js
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

test('increments count', () => {
  const { result } = renderHook(() => useCounter());
  
  expect(result.current.count).toBe(0);
  
  act(() => {
    result.current.increment();
  });
  
  expect(result.current.count).toBe(1);
});

test('starts with initial value', () => {
  const { result } = renderHook(() => useCounter(10));
  expect(result.current.count).toBe(10);
});

test('resets to initial value', () => {
  const { result } = renderHook(() => useCounter(5));
  
  act(() => {
    result.current.increment();
    result.current.increment();
  });
  
  expect(result.current.count).toBe(7);
  
  act(() => {
    result.current.reset();
  });
  
  expect(result.current.count).toBe(5);
});
```

---

### 7. **Integration Tests vs. Unit Tests**

**Unit test:** Test a single component in isolation.

```jsx
// Unit test: Button component alone
test('button calls onClick when clicked', async () => {
  const handleClick = jest.fn();
  render(<Button onClick={handleClick}>Click me</Button>);
  
  await userEvent.click(screen.getByRole('button'));
  
  expect(handleClick).toHaveBeenCalledTimes(1);
});
```

**Integration test:** Test multiple components working together.

```jsx
// Integration test: Whole TodoApp
test('can add and remove todos', async () => {
  render(<TodoApp />);
  const user = userEvent.setup();
  
  // Add a todo
  const input = screen.getByPlaceholderText(/add todo/i);
  await user.type(input, 'Buy milk{Enter}');
  
  expect(screen.getByText('Buy milk')).toBeInTheDocument();
  
  // Remove the todo
  const deleteButton = screen.getByRole('button', { name: /delete/i });
  await user.click(deleteButton);
  
  expect(screen.queryByText('Buy milk')).not.toBeInTheDocument();
});
```

**Best practice:** Prefer integration tests (test user workflows) over unit tests (test individual components).

---

## Common Pitfalls

### ❌ **Pitfall 1: Not cleaning up after tests**

```jsx
// BAD: Mocks leak between tests
beforeAll(() => {
  global.fetch = jest.fn();  // Never cleaned up!
});

test('test 1', () => { /* ... */ });
test('test 2', () => { /* ... */ });  // Still using mocked fetch
```

**Fix:**

```jsx
beforeEach(() => {
  global.fetch = jest.fn();
});

afterEach(() => {
  jest.restoreAllMocks();  // Clean up
});
```

---

### ❌ **Pitfall 2: Not using screen**

```jsx
// BAD: Using destructured queries
test('bad example', () => {
  const { getByText } = render(<App />);
  expect(getByText('Hello')).toBeInTheDocument();
});

// GOOD: Using screen
test('good example', () => {
  render(<App />);
  expect(screen.getByText('Hello')).toBeInTheDocument();
});
```

**Why screen is better:** Cleaner code, no need to destructure, works with debugging tools.

---

### ❌ **Pitfall 3: Forgetting to await async queries**

```jsx
// BAD: findBy returns a promise, must await
test('bad async test', () => {
  render(<AsyncComponent />);
  const element = screen.findByText('Loaded');  // Promise!
  expect(element).toBeInTheDocument();  // FAILS
});

// GOOD
test('good async test', async () => {
  render(<AsyncComponent />);
  const element = await screen.findByText('Loaded');
  expect(element).toBeInTheDocument();
});
```

---

### ❌ **Pitfall 4: Testing implementation details**

```jsx
// BAD: Testing class names (implementation detail)
test('bad test', () => {
  render(<Button />);
  const button = screen.getByRole('button');
  expect(button).toHaveClass('btn-primary');  // Fragile!
});

// GOOD: Testing behavior
test('good test', () => {
  const handleClick = jest.fn();
  render(<Button onClick={handleClick}>Click</Button>);
  
  fireEvent.click(screen.getByRole('button'));
  
  expect(handleClick).toHaveBeenCalled();
});
```

---

## Quick Self-Check

✅ **You understand React testing if you can:**

1. Explain the difference between getBy, queryBy, and findBy
2. Write a test for a form submission
3. Mock an API call and test async data fetching
4. Test user interactions (typing, clicking, selecting)
5. Explain why testing implementation details is bad
6. Test a custom hook
7. Use waitFor for async assertions
8. Mock a module with jest.mock()
9. Write an integration test for a multi-step workflow
10. Explain when to use fireEvent vs. userEvent

---

## Interview Questions to Practice

**Beginner:**
1. What's the difference between unit tests and integration tests?
2. What does RTL's `screen` object do?
3. Why do we use `getByRole` instead of `getByTestId`?

**Intermediate:**
4. How do you test a component that fetches data on mount?
5. What's the difference between `getBy`, `queryBy`, and `findBy`?
6. How do you mock a module in Jest?
7. How would you test a custom hook?

**Advanced:**
8. How would you test a component with debounced input?
9. Explain how to test a component that uses Context
10. How would you structure tests for a large application?

---

## Further Practice

**Build and test:**
1. **Todo app** with add/remove/complete functionality
2. **Search component** with debounced API calls
3. **Login form** with validation and error handling
4. **Infinite scroll** component
5. **Modal/Dialog** with focus trapping
6. **Multi-step wizard** with validation on each step

**Resources:**
- [React Testing Library docs](https://testing-library.com/react)
- [Jest docs](https://jestjs.io/)
- [Common Testing Library mistakes](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library)
- [Testing Playground](https://testing-playground.com/) - Practice queries

---

## Summary

**Testing philosophy:**
- Test behavior, not implementation
- Tests should resemble how users interact with your app
- Prefer integration tests over isolated unit tests

**Query priority:**
1. getByRole (accessible)
2. getByLabelText (forms)
3. getByText (content)
4. getByTestId (last resort)

**Key tools:**
- **Jest:** Test runner, assertions, mocks
- **React Testing Library:** User-centric queries and utilities
- **userEvent:** Realistic user interactions
- **waitFor / findBy:** Async testing

**What to test:**
- User interactions (clicking, typing)
- Conditional rendering (loading, error, success states)
- Form submission and validation
- API integration (with mocked responses)
- Edge cases and error handling

**What NOT to test:**
- Implementation details (class names, state variable names)
- Third-party libraries (they have their own tests)
- Trivial code (simple prop passing)

**Remember:** Good tests give you confidence to refactor and catch real bugs. Bad tests break on every change and catch nothing.
