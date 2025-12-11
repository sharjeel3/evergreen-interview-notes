# Hooks Basics

**Level:** 🏁 Beginner  
**Tags:** `hooks`, `useState`, `useEffect`, `rules-of-hooks`, `interview`, `fundamentals`

---

## Why this matters

Hooks revolutionized React. In interviews, you must:

- explain **why hooks exist** (not just how to use them)
- know the **Rules of Hooks**
- understand the most common hooks
- recognize when to create custom hooks

**Interview red flag:** breaking Rules of Hooks or not knowing why they exist

---

## Core Ideas

### 1. **Hooks let function components have state and side effects**

Before hooks (React < 16.8):
- Function components = "stateless" / "dumb"
- Need class components for state/lifecycle

**After hooks:**
- Function components can do everything
- Classes are legacy

**Mental model:**  
"Hooks are functions that 'hook into' React features from function components."

---

### 2. **Rules of Hooks (CRITICAL)**

**Rule 1: Only call hooks at the top level**

```jsx
// ❌ Conditional hooks break React's internal state tracking
function Bad({ condition }) {
  if (condition) {
    const [state, setState] = useState(0);  // ERROR!
  }
}

// ✅ Always call hooks in the same order
function Good({ condition }) {
  const [state, setState] = useState(0);
  if (condition) {
    // Use state here
  }
}
```

**Rule 2: Only call hooks from React functions**

```jsx
// ❌ Can't call hooks in regular JS functions
function regularFunction() {
  const [state, setState] = useState(0);  // ERROR!
}

// ✅ Call from React components or custom hooks
function MyComponent() {
  const [state, setState] = useState(0);  // OK
}

function useMyHook() {
  const [state, setState] = useState(0);  // OK (custom hook)
}
```

**Why?** React relies on call order to match hooks with state.

---

### 3. **useState — state in function components**

```jsx
const [value, setValue] = useState(initialValue);
```

- `value`: current state
- `setValue`: function to update state
- `initialValue`: initial state (or function returning initial state)

**Key:** Each `useState` call creates independent state.

---

### 4. **useEffect — side effects after render**

```jsx
useEffect(() => {
  // Side effect code here
  
  return () => {
    // Cleanup (optional)
  };
}, [dependencies]);
```

**Runs:**
- After every render (if no deps)
- After first render (if deps = `[]`)
- After render when deps change (if deps = `[x, y]`)

**Mental model:**  
"useEffect = 'do something after React updates the DOM'."

---

### 5. **Built-in hooks overview**

| Hook | Purpose |
|------|---------|
| `useState` | Component state |
| `useEffect` | Side effects (fetch, subscriptions, timers) |
| `useContext` | Access context values |
| `useReducer` | Complex state logic (like Redux) |
| `useCallback` | Memoize callbacks |
| `useMemo` | Memoize expensive calculations |
| `useRef` | Persist values / access DOM |
| `useImperativeHandle` | Customize ref exposed to parent |
| `useLayoutEffect` | Sync effect before paint |
| `useDebugValue` | Custom hook debugging |

---

## Examples

### Example 1 — useState

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}
```

---

### Example 2 — useEffect (data fetching)

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    // Fetch user when userId changes
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => setUser(data));
  }, [userId]);  // Re-run when userId changes
  
  if (!user) return <div>Loading...</div>;
  return <div>{user.name}</div>;
}
```

---

### Example 3 — useEffect (subscription with cleanup)

```jsx
function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();
    
    // Cleanup function (runs on unmount or before next effect)
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
  
  return <div>Chat Room {roomId}</div>;
}
```

**Key:** Return cleanup function to avoid memory leaks.

---

### Example 4 — Multiple hooks in one component

```jsx
function UserDashboard({ userId }) {
  // Multiple state variables
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(true);
  
  // Multiple effects
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);
  
  useEffect(() => {
    fetchPosts(userId).then(setPosts);
  }, [userId]);
  
  useEffect(() => {
    if (user && posts.length > 0) {
      setLoading(false);
    }
  }, [user, posts]);
  
  if (loading) return <div>Loading...</div>;
  return <div>{/* Render user and posts */}</div>;
}
```

---

### Example 5 — useRef (persisting values without re-render)

```jsx
function Timer() {
  const [seconds, setSeconds] = useState(0);
  const intervalRef = useRef(null);
  
  const start = () => {
    intervalRef.current = setInterval(() => {
      setSeconds(s => s + 1);
    }, 1000);
  };
  
  const stop = () => {
    clearInterval(intervalRef.current);
  };
  
  return (
    <div>
      <p>Seconds: {seconds}</p>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
    </div>
  );
}
```

**Key:** `useRef` doesn't trigger re-render when changed.

---

## Common Pitfalls

### 1. **Breaking Rules of Hooks**

```jsx
// ❌ Conditional hook call
function Bad({ condition }) {
  if (condition) {
    const [state, setState] = useState(0);
  }
}

// ❌ Hook in loop
function Bad({ items }) {
  items.forEach(item => {
    const [state, setState] = useState(0);
  });
}

// ❌ Hook in callback
function Bad() {
  const handleClick = () => {
    const [state, setState] = useState(0);
  };
}
```

**ESLint plugin `eslint-plugin-react-hooks` catches these.**

---

### 2. **Missing effect dependencies**

```jsx
function Bad() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      console.log(count);  // Stale closure!
    }, 1000);
    
    return () => clearInterval(interval);
  }, []);  // ❌ Missing count in deps
}

// ✅ Add all dependencies
useEffect(() => {
  const interval = setInterval(() => {
    console.log(count);
  }, 1000);
  
  return () => clearInterval(interval);
}, [count]);

// OR use functional update
useEffect(() => {
  const interval = setInterval(() => {
    setCount(c => c + 1);
  }, 1000);
  
  return () => clearInterval(interval);
}, []);
```

---

### 3. **Infinite effect loops**

```jsx
// ❌ Infinite loop (state update → re-render → effect → state update → ...)
function Bad() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    setCount(count + 1);  // Triggers effect again
  });  // No dependency array!
}

// ✅ Use dependency array
useEffect(() => {
  // Only runs once
  setCount(1);
}, []);
```

---

### 4. **Not cleaning up effects**

```jsx
// ❌ Memory leak (subscription never cleaned up)
function Bad() {
  useEffect(() => {
    const subscription = api.subscribe();
  }, []);
}

// ✅ Return cleanup function
function Good() {
  useEffect(() => {
    const subscription = api.subscribe();
    return () => subscription.unsubscribe();
  }, []);
}
```

---

### 5. **Using async functions directly in useEffect**

```jsx
// ❌ Can't use async directly
useEffect(async () => {
  const data = await fetchData();
}, []);

// ✅ Define async function inside
useEffect(() => {
  const loadData = async () => {
    const data = await fetchData();
    setData(data);
  };
  loadData();
}, []);

// ✅ Or use IIFE
useEffect(() => {
  (async () => {
    const data = await fetchData();
    setData(data);
  })();
}, []);
```

---

## FAQ

**Q: Why can't I call hooks conditionally?**  
A: React tracks hooks by call order. Conditional calls break this tracking.

**Q: What's the difference between `useEffect` and `useLayoutEffect`?**  
A: `useEffect` runs after paint (async). `useLayoutEffect` runs before paint (sync).  
Use `useLayoutEffect` when you need to read layout or make DOM changes synchronously.

**Q: When should I create a custom hook?**  
A: When you have reusable stateful logic across components.

**Q: Can I use hooks in class components?**  
A: No. Hooks only work in function components.

**Q: Why does my effect run twice in development?**  
A: React 18 Strict Mode intentionally runs effects twice to catch bugs.

---

## Quick Self-Check

1. What's wrong with this code?
   ```jsx
   if (condition) {
     const [state, setState] = useState(0);
   }
   ```

2. What does this effect do?
   ```jsx
   useEffect(() => {
     console.log('Rendered');
   }, []);
   ```
   (Runs once after initial render)

3. When do you need cleanup functions?
   - Subscriptions, timers, event listeners

4. What's the difference between these?
   ```jsx
   useEffect(() => { ... });        // Runs after every render
   useEffect(() => { ... }, []);    // Runs once (mount)
   useEffect(() => { ... }, [x]);   // Runs when x changes
   ```

5. Can you call hooks inside loops, conditions, or nested functions?
   - **No!** Only at top level of function component or custom hook.

---

## Further Practice

1. **Convert:** Take class component and convert to function + hooks
2. **Build:** Data fetching hook (`useFetch`)
3. **Debug:** Create effect with missing dependencies and observe stale closures
4. **Experiment:** Try breaking Rules of Hooks and understand ESLint errors
5. **Read:** [Official Hooks FAQ](https://react.dev/reference/react)

---

## Key Takeaways

- Hooks let function components use state and effects
- **Rules of Hooks:** Only at top level, only in React functions
- `useState` for state, `useEffect` for side effects
- Always include dependencies in `useEffect` array
- Return cleanup function from effects to avoid leaks
- Multiple hooks can be used in one component
- ESLint plugin helps enforce Rules of Hooks
- Hooks replaced class components as the standard
