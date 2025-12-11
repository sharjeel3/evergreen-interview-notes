# useEffect & Side Effects

**Level:** 🚦 Intermediate  
**Tags:** `useEffect`, `side-effects`, `lifecycle`, `interview`, `hooks`

---

## Why this matters

`useEffect` is arguably the most misunderstood React hook and a top interview topic because it's where most bugs occur in production React applications. Mastering useEffect demonstrates you understand React's rendering lifecycle, asynchronous behavior, and can handle real-world complexities like data fetching, subscriptions, and cleanup.

**Why interviewers care:**
- useEffect bugs (infinite loops, memory leaks, stale closures) are the #1 source of production issues in React apps
- Understanding dependencies reveals whether you grasp React's reactivity model and how data flows
- Cleanup functions separate developers who write robust code from those who create memory leaks
- useEffect questions test asynchronous thinking—critical for handling API calls, timers, and subscriptions
- Senior developers must debug useEffect issues across large codebases and mentor others on proper usage

**Real-world implications:**
- **Data fetching:** Almost every app fetches data on mount or when parameters change—mishandled effects cause race conditions
- **Memory leaks:** Missing cleanup for subscriptions/timers/listeners causes apps to slow down over time
- **Performance:** Incorrect dependencies cause unnecessary API calls, expensive recalculations, and poor UX
- **Stale closures:** The most common useEffect bug—effects capture old values when dependencies are missing
- **Race conditions:** Fast user interactions can cause effects to overlap, showing stale data
- **Debugging complexity:** useEffect issues are hard to debug because effects run asynchronously after render

**Common real-world scenarios:**
- Fetching user data when route parameters change
- Setting up WebSocket connections for real-time features
- Subscribing to observables or event emitters
- Synchronizing with browser APIs (localStorage, geolocation)
- Implementing timers, animations, or polling
- Setting document title or managing global state

**What you must know:**
- **Side effects:** Operations outside React's rendering (fetch, subscribe, timers, DOM manipulation, logging)
- **Dependency array:** Controls when effects run—`[]` (once), `[a, b]` (when a/b change), none (every render)
- **Cleanup functions:** Prevent memory leaks by unsubscribing/clearing timers/removing listeners
- **Timing:** useEffect runs *after* render (asynchronous), doesn't block painting
- **Stale closures:** Why missing dependencies cause bugs (effect captures old variable values)
- **Async patterns:** Can't use async directly, must define async function inside effect
- **Race conditions:** How to handle overlapping async operations (cleanup flags, AbortController)
- **useEffect vs useLayoutEffect:** When to use synchronous effects (rare)

**Interview red flag:** Not understanding the dependency array or cleanup functions shows you'll write buggy code in production. Missing dependencies cause stale closures (shows old data), and missing cleanup causes memory leaks that crash apps over time.

---

## Core Ideas

### 1. **Side effects = operations outside React's rendering**

Side effects include:
- Data fetching (API calls)
- Subscriptions (WebSockets, event listeners)
- Timers (setTimeout, setInterval)
- DOM manipulation
- Logging, analytics

**Mental model:**  
"useEffect = 'after React renders, do this side thing'."

---

### 2. **Dependency array controls when effect runs**

```jsx
useEffect(() => {
  // Effect code
}, [/* dependencies */]);
```

| Dependency Array | When Effect Runs |
|------------------|------------------|
| `undefined` (no array) | After **every** render |
| `[]` (empty array) | **Once** after initial render |
| `[a, b]` | After renders where `a` or `b` changed |

---

### 3. **Cleanup function prevents memory leaks**

```jsx
useEffect(() => {
  // Setup
  const subscription = api.subscribe();
  
  // Cleanup (runs before next effect or unmount)
  return () => {
    subscription.unsubscribe();
  };
}, []);
```

**When cleanup runs:**
- Before effect runs again (if deps changed)
- When component unmounts

---

### 4. **Effects run AFTER render (asynchronous)**

```jsx
function Component() {
  console.log('1. Render');
  
  useEffect(() => {
    console.log('3. Effect');
  });
  
  console.log('2. Still rendering');
}

// Output: 1 → 2 → 3
```

**Key:** Effects don't block rendering.

---

### 5. **Common effect patterns**

| Pattern | Example |
|---------|---------|
| Fetch on mount | `useEffect(() => { fetch... }, [])` |
| Subscribe/unsubscribe | `useEffect(() => { sub(); return unsub; }, [])` |
| Update on prop change | `useEffect(() => { ... }, [propA])` |
| Sync with external system | `useEffect(() => { syncState(); }, [state])` |

---

## Examples

### Example 1 — Fetch data on mount

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    setLoading(true);
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        setUser(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err);
        setLoading(false);
      });
  }, [userId]);  // Re-fetch when userId changes
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  return <div>{user.name}</div>;
}
```

---

### Example 2 — Subscription with cleanup

```jsx
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  
  useEffect(() => {
    // Setup: connect to chat room
    const connection = chatAPI.connect(roomId);
    
    connection.on('message', (msg) => {
      setMessages(prev => [...prev, msg]);
    });
    
    // Cleanup: disconnect when roomId changes or component unmounts
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
  
  return (
    <div>
      {messages.map(msg => <div key={msg.id}>{msg.text}</div>)}
    </div>
  );
}
```

---

### Example 3 — Timer with cleanup

```jsx
function Timer() {
  const [seconds, setSeconds] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds(s => s + 1);  // Functional update avoids stale closure
    }, 1000);
    
    // Cleanup: clear interval on unmount
    return () => clearInterval(interval);
  }, []);  // Empty deps = setup once, cleanup on unmount
  
  return <div>Seconds: {seconds}</div>;
}
```

---

### Example 4 — Event listener with cleanup

```jsx
function WindowSize() {
  const [width, setWidth] = useState(window.innerWidth);
  
  useEffect(() => {
    const handleResize = () => setWidth(window.innerWidth);
    
    window.addEventListener('resize', handleResize);
    
    // Cleanup: remove listener
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  return <div>Window width: {width}px</div>;
}
```

---

### Example 5 — Async function in effect

```jsx
function UserData({ userId }) {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    // ❌ Can't make useEffect callback async directly
    // useEffect(async () => { ... })  // ERROR!
    
    // ✅ Define async function inside
    const fetchData = async () => {
      const response = await fetch(`/api/users/${userId}`);
      const json = await response.json();
      setData(json);
    };
    
    fetchData();
  }, [userId]);
  
  return <div>{data?.name}</div>;
}

// ✅ Alternative: IIFE
useEffect(() => {
  (async () => {
    const response = await fetch(`/api/users/${userId}`);
    const json = await response.json();
    setData(json);
  })();
}, [userId]);
```

---

## Common Pitfalls

### 1. **Missing dependencies (stale closures)**

```jsx
// ❌ Stale closure: always logs 0
function Counter() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      console.log(count);  // Always 0!
    }, 1000);
    
    return () => clearInterval(interval);
  }, []);  // Missing count in deps
}

// ✅ Include count in deps
useEffect(() => {
  const interval = setInterval(() => {
    console.log(count);  // Current value
  }, 1000);
  
  return () => clearInterval(interval);
}, [count]);  // Re-create interval when count changes

// ✅ Or use functional update
useEffect(() => {
  const interval = setInterval(() => {
    setCount(c => {
      console.log(c);  // Current value
      return c + 1;
    });
  }, 1000);
  
  return () => clearInterval(interval);
}, []);  // No dependencies needed
```

---

### 2. **Infinite loops**

```jsx
// ❌ Infinite loop: state update → re-render → effect → state update → ...
function Bad() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    setCount(count + 1);  // Causes re-render
  });  // No deps = runs after every render
}

// ✅ Use dependency array
useEffect(() => {
  setCount(1);
}, []);  // Runs once

// ❌ Object/array in deps (always different reference)
useEffect(() => {
  fetchData(options);
}, [options]);  // New object every render!

// ✅ Destructure or use primitive values
const { id, filter } = options;
useEffect(() => {
  fetchData(options);
}, [id, filter]);
```

---

### 3. **Not cleaning up**

```jsx
// ❌ Memory leak: subscription never removed
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

### 4. **Race conditions in async effects**

```jsx
// ❌ Race condition: fast userId change may show old data
function Bad({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);
}

// ✅ Use cleanup to ignore stale responses
function Good({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    let ignore = false;
    
    fetchUser(userId).then(data => {
      if (!ignore) setUser(data);
    });
    
    return () => { ignore = true; };
  }, [userId]);
}

// ✅ Alternative: AbortController
useEffect(() => {
  const controller = new AbortController();
  
  fetch(`/api/users/${userId}`, { signal: controller.signal })
    .then(res => res.json())
    .then(setUser)
    .catch(err => {
      if (err.name !== 'AbortError') throw err;
    });
  
  return () => controller.abort();
}, [userId]);
```

---

### 5. **Setting state with non-primitive deps**

```jsx
// ❌ Object changes reference every render
function Bad({ options }) {
  useEffect(() => {
    fetchData(options);
  }, [options]);  // Runs every render!
}

// ✅ Destructure primitive values
function Good({ options }) {
  const { id, filter } = options;
  useEffect(() => {
    fetchData({ id, filter });
  }, [id, filter]);
}

// ✅ Or use useMemo
const options = useMemo(() => ({ id, filter }), [id, filter]);
```

---

## FAQ

**Q: When should I clean up an effect?**  
A: Whenever the effect creates a subscription, timer, or external resource.

**Q: Can I use async/await in useEffect?**  
A: Not directly. Define async function inside or use IIFE.

**Q: What if I want to run effect only once?**  
A: Use empty dependency array: `useEffect(() => {...}, [])`

**Q: Why is my effect running twice in development?**  
A: React 18 Strict Mode intentionally runs effects twice to catch bugs.

**Q: What's the difference between useEffect and useLayoutEffect?**  
A: `useEffect` runs after paint (async). `useLayoutEffect` runs before paint (sync).  
Use `useLayoutEffect` for DOM measurements or synchronous DOM updates.

**Q: How do I fetch data on prop change?**  
A: Include prop in dependency array: `useEffect(() => {...}, [propName])`

---

## Quick Self-Check

1. What's wrong with this?
   ```jsx
   useEffect(() => {
     setCount(count + 1);
   });
   ```
   (Infinite loop: no deps array)

2. When does cleanup function run?
   - Before next effect
   - On unmount

3. How do you run effect only once?
   ```jsx
   useEffect(() => {...}, []);
   ```

4. What's a stale closure?
   (Effect captures old variable value when deps are missing)

5. How do you cancel fetch on unmount?
   (Use AbortController or ignore flag)

---

## Further Practice

1. **Build:** Data fetching hook with loading/error states
2. **Build:** useInterval custom hook
3. **Debug:** Create and fix stale closure bugs
4. **Optimize:** Refactor class lifecycle methods to useEffect
5. **Experiment:** Create race condition and fix with cleanup

---

## Key Takeaways

- useEffect runs side effects after render
- Dependency array controls when effect runs: `[]`, `[a]`, or none
- Return cleanup function for subscriptions, timers, listeners
- Effects are asynchronous (don't block rendering)
- Missing dependencies cause stale closures
- Empty deps `[]` = run once (mount + unmount cleanup)
- Use functional updates to avoid stale state
- Handle race conditions with cleanup/AbortController
- React 18 Strict Mode runs effects twice in dev
