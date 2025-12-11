# Custom Hooks

**Level:** 🚦 Intermediate  
**Tags:** `custom-hooks`, `reusability`, `abstraction`, `interview`, `hooks`

---

## Why this matters

Custom hooks are one of React's most powerful features for code reuse and abstraction, yet they're frequently misunderstood or underutilized. Interviewers assess custom hooks to evaluate your ability to create clean abstractions, recognize reusable patterns, and write maintainable code that scales across large applications.

**Why interviewers care:**
- Custom hooks reveal whether you can extract reusable logic vs copying code everywhere
- Creating good abstractions is a key skill that separates senior from junior developers
- Hook composition demonstrates understanding of React's functional programming model
- Interviewers often ask you to refactor messy components into clean custom hooks on the spot

**Real-world implications:**
- **Code reuse:** Custom hooks eliminate duplicate logic across components (DRY principle)
- **Maintainability:** Changes to shared logic happen in one place, not scattered across files
- **Testing:** Isolated hooks are easier to test than component-embedded logic
- **Team productivity:** Well-designed hooks become shared utilities that speed up development
- **Separation of concerns:** Business logic separated from UI concerns makes code clearer
- **Composition:** Small, focused hooks can be combined to create complex functionality

**Common scenarios requiring custom hooks:**
- Data fetching with loading/error states (useFetch, useQuery)
- Form handling and validation (useForm, useInput)
- Browser APIs (useLocalStorage, useGeolocation, useMediaQuery)
- Event listeners (useEventListener, useClickOutside)
- Timers and intervals (useInterval, useTimeout, useDebounce)
- Previous values (usePrevious)
- Async operations (useAsync, usePromise)

**What you must know:**
- **Naming:** Custom hooks MUST start with "use" (enforced by linter)
- **Rules of Hooks:** Custom hooks follow same rules as built-in hooks
- **Composition:** Custom hooks can call other hooks (built-in or custom)
- **Return values:** Can return anything: values, arrays, objects, functions
- **Parameters:** Accept any parameters like regular functions
- **When to create:** When you see repeated logic with state/effects across components

**Interview red flag:** Not starting custom hook names with "use" (breaks convention and linting), or creating overly complex hooks that do too many things (violates single responsibility principle). Also bad: not knowing when to extract a custom hook vs keeping logic inline.

---

## Core Ideas

### 1. **Custom hooks extract reusable stateful logic**

```jsx
// ❌ Before: Duplicate logic in every component
function ComponentA() {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetch('/api/data').then(res => res.json()).then(setData);
  }, []);
}

function ComponentB() {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetch('/api/data').then(res => res.json()).then(setData);
  }, []);
}

// ✅ After: Reusable custom hook
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetch(url).then(res => res.json()).then(setData);
  }, [url]);
  
  return { data, loading };
}

// Now both components use the hook
function ComponentA() {
  const { data, loading } = useFetch('/api/data');
}
```

**Mental model:**  
"Custom hooks = functions that use hooks inside them."

---

### 2. **Custom hooks must follow Rules of Hooks**

```jsx
// ❌ Can't call hooks conditionally
function useBadHook(condition) {
  if (condition) {
    const [state, setState] = useState(0);  // ERROR!
  }
}

// ✅ Hooks at top level always
function useGoodHook(condition) {
  const [state, setState] = useState(0);
  
  if (condition) {
    // Use state here
  }
}
```

---

### 3. **Each component gets independent state**

```jsx
function useCounter() {
  const [count, setCount] = useState(0);
  const increment = () => setCount(c => c + 1);
  return { count, increment };
}

function App() {
  const counterA = useCounter();  // Independent state
  const counterB = useCounter();  // Independent state
  
  // counterA.count and counterB.count are separate!
}
```

**Key:** Hooks don't share state between components—each call creates new state.

---

### 4. **Custom hooks can compose other hooks**

```jsx
function useUser(userId) {
  // Uses built-in hooks
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);
  
  return user;
}

function useUserWithPosts(userId) {
  // Uses custom hook + built-in hooks
  const user = useUser(userId);
  const [posts, setPosts] = useState([]);
  
  useEffect(() => {
    if (user) {
      fetchPosts(user.id).then(setPosts);
    }
  }, [user]);
  
  return { user, posts };
}
```

---

### 5. **Return patterns**

```jsx
// Pattern 1: Object (named returns)
function useAuth() {
  return { user, login, logout, loading };
}
const { user, login } = useAuth();

// Pattern 2: Array (positional returns, like useState)
function useToggle(initial = false) {
  const [value, setValue] = useState(initial);
  const toggle = () => setValue(v => !v);
  return [value, toggle];
}
const [isOpen, toggleOpen] = useToggle();

// Pattern 3: Single value
function useWindowWidth() {
  const [width, setWidth] = useState(window.innerWidth);
  // ... effect code
  return width;
}
const width = useWindowWidth();
```

---

## Examples

### Example 1 — useFetch (data fetching)

```jsx
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    let ignore = false;
    
    setLoading(true);
    fetch(url)
      .then(res => res.json())
      .then(data => {
        if (!ignore) {
          setData(data);
          setLoading(false);
        }
      })
      .catch(err => {
        if (!ignore) {
          setError(err);
          setLoading(false);
        }
      });
    
    return () => { ignore = true; };
  }, [url]);
  
  return { data, loading, error };
}

// Usage
function UserProfile({ userId }) {
  const { data: user, loading, error } = useFetch(`/api/users/${userId}`);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  return <div>{user.name}</div>;
}
```

---

### Example 2 — useLocalStorage (persistent state)

```jsx
function useLocalStorage(key, initialValue) {
  // State stored in localStorage
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });
  
  // Return wrapped version that persists to localStorage
  const setValue = (value) => {
    try {
      // Allow value to be function (like useState)
      const valueToStore = value instanceof Function 
        ? value(storedValue) 
        : value;
      
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(error);
    }
  };
  
  return [storedValue, setValue];
}

// Usage
function App() {
  const [name, setName] = useLocalStorage('name', 'Anonymous');
  
  return (
    <input 
      value={name} 
      onChange={e => setName(e.target.value)} 
    />
  );
}
```

---

### Example 3 — useDebounce (performance optimization)

```jsx
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);
  
  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    // Cleanup timeout if value changes before delay
    return () => clearTimeout(handler);
  }, [value, delay]);
  
  return debouncedValue;
}

// Usage: Search input that doesn't trigger API on every keystroke
function SearchBox() {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearch = useDebounce(searchTerm, 500);
  
  useEffect(() => {
    if (debouncedSearch) {
      // API call only happens 500ms after user stops typing
      searchAPI(debouncedSearch);
    }
  }, [debouncedSearch]);
  
  return (
    <input 
      value={searchTerm}
      onChange={e => setSearchTerm(e.target.value)}
      placeholder="Search..."
    />
  );
}
```

---

### Example 4 — useToggle (boolean state helper)

```jsx
function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);
  
  const toggle = useCallback(() => {
    setValue(v => !v);
  }, []);
  
  const setTrue = useCallback(() => setValue(true), []);
  const setFalse = useCallback(() => setValue(false), []);
  
  return [value, toggle, setTrue, setFalse];
}

// Usage
function Modal() {
  const [isOpen, toggle, open, close] = useToggle();
  
  return (
    <>
      <button onClick={open}>Open Modal</button>
      {isOpen && (
        <div className="modal">
          <button onClick={close}>Close</button>
        </div>
      )}
    </>
  );
}
```

---

### Example 5 — usePrevious (track previous value)

```jsx
function usePrevious(value) {
  const ref = useRef();
  
  useEffect(() => {
    ref.current = value;
  }, [value]);
  
  return ref.current;
}

// Usage: Compare current vs previous value
function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);
  
  return (
    <div>
      <p>Current: {count}</p>
      <p>Previous: {prevCount}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

---

## Common Pitfalls

### 1. **Not starting with "use"**

```jsx
// ❌ Doesn't follow convention
function fetchData(url) {
  const [data, setData] = useState(null);  // ESLint error!
  // ...
}

// ✅ Must start with "use"
function useFetchData(url) {
  const [data, setData] = useState(null);
  // ...
}
```

---

### 2. **Sharing state between components (misunderstanding)**

```jsx
// Custom hooks DON'T share state!
function useCounter() {
  const [count, setCount] = useState(0);
  return { count, setCount };
}

function ComponentA() {
  const { count } = useCounter();  // count = 0
}

function ComponentB() {
  const { count } = useCounter();  // count = 0 (different state!)
}

// For shared state, use Context or state management
```

---

### 3. **Overly complex hooks**

```jsx
// ❌ Hook doing too many things
function useEverything() {
  const user = useUser();
  const posts = usePosts();
  const comments = useComments();
  const likes = useLikes();
  // ... 50 more lines
  return { user, posts, comments, likes };
}

// ✅ Small, focused hooks
function useUser() { /* ... */ }
function usePosts() { /* ... */ }
function useComments() { /* ... */ }
```

**Principle:** Single responsibility—each hook does one thing well.

---

### 4. **Missing dependencies in effects**

```jsx
// ❌ Stale closure in custom hook
function useFetch(url) {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetch(url).then(res => res.json()).then(setData);
  }, []);  // Missing url dependency!
  
  return data;
}

// ✅ Include all dependencies
useEffect(() => {
  fetch(url).then(res => res.json()).then(setData);
}, [url]);
```

---

### 5. **Not cleaning up effects**

```jsx
// ❌ Memory leak
function useInterval(callback, delay) {
  useEffect(() => {
    const id = setInterval(callback, delay);
    // Missing cleanup!
  }, [callback, delay]);
}

// ✅ Always cleanup
function useInterval(callback, delay) {
  useEffect(() => {
    const id = setInterval(callback, delay);
    return () => clearInterval(id);
  }, [callback, delay]);
}
```

---

## FAQ

**Q: When should I create a custom hook?**  
A: When you see repeated logic with state/effects in 2+ components. Or when complex logic makes a component hard to read.

**Q: Can custom hooks call other custom hooks?**  
A: Yes! Hooks can compose freely: `useA` can call `useB`, which calls `useC`.

**Q: Do custom hooks share state?**  
A: No. Each component that calls a hook gets independent state. For shared state, use Context.

**Q: What should custom hooks return?**  
A: Whatever makes sense: object, array, single value, or functions. Follow patterns that make usage clear.

**Q: Can I use hooks conditionally in custom hooks?**  
A: No. Same Rules of Hooks apply: hooks must be called in same order every render.

**Q: How do I test custom hooks?**  
A: Use `@testing-library/react-hooks` or test through components that use them.

**Q: Should I always use custom hooks?**  
A: No. If logic is used once, keep it inline. Extract when you see repetition or complexity.

---

## Quick Self-Check

1. What must custom hook names start with?
   (`use` prefix)

2. What's wrong with this?
   ```jsx
   function fetchData() {
     const [data, setData] = useState(null);
   }
   ```
   (Doesn't start with "use")

3. Do components sharing a custom hook share state?
   (No, each gets independent state)

4. Can custom hooks call other hooks?
   (Yes, hooks can compose freely)

5. When should you extract a custom hook?
   (Repeated logic with state/effects, or to reduce complexity)

---

## Further Practice

1. **Build:** `useAsync` hook for any async operation with loading/error
2. **Build:** `useForm` hook with validation and submission handling
3. **Build:** `useMediaQuery` for responsive design
4. **Build:** `useClickOutside` for closing dropdowns/modals
5. **Refactor:** Find repeated logic in existing code and extract custom hooks
6. **Study:** Browse popular hook libraries (react-use, usehooks-ts)

---

## Key Takeaways

- Custom hooks extract reusable stateful logic from components
- MUST start with "use" prefix (convention + linting requirement)
- Follow Rules of Hooks (top level, same order every render)
- Each component gets independent state from hook
- Hooks can compose other hooks (built-in or custom)
- Return objects, arrays, or values depending on use case
- Keep hooks focused (single responsibility)
- Include all dependencies in effects
- Always clean up effects (timers, subscriptions, listeners)
- Extract when you see repetition or need to reduce complexity
