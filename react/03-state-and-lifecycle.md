# State & Lifecycle

**Level:** 🏁 Beginner  
**Tags:** `state`, `useState`, `lifecycle`, `interview`, `fundamentals`

---

## Why this matters

State is the mechanism that makes React applications interactive and dynamic. While props let you pass data down, state enables components to respond to user input, server responses, and time-based changes. Mastery of state management is critical because it's central to almost every React interview and real-world application.

**Why interviewers care:**
- State management separates junior developers (who can read docs) from experienced ones (who understand trade-offs)
- How you handle state reveals your understanding of React's rendering cycle and performance implications
- State bugs are among the most common production issues (race conditions, stale data, unnecessary re-renders)
- Complex applications need sophisticated state strategies, and interviewers want to know you can scale beyond basic useState

**Real-world implications:**
- **User experience:** State updates drive UI changes—bugs here directly impact users
- **Performance:** Unnecessary state updates cause expensive re-renders and sluggish UIs
- **Data integrity:** Improper state mutations can cause data corruption and hard-to-debug issues
- **Asynchronous complexity:** State updates are batched and asynchronous, leading to common pitfalls
- **Testing:** Stateful components require careful testing strategies
- **Architecture:** State placement (local vs lifted vs global) affects application maintainability

**What you must know:**
- State is mutable, internal data that triggers re-renders when changed
- The difference between props (external, immutable) and state (internal, mutable)
- How to update state correctly using setter functions (never direct mutation)
- State updates are asynchronous and batched for performance
- When to use functional updates (`setState(prev => prev + 1)`)
- How to structure state (multiple useState vs object state)
- When to lift state up to parent components

**Interview red flag:** Mutating state directly (`count = count + 1` or `user.name = "Bob"`) shows a fundamental misunderstanding that will cause bugs in production. React relies on immutability to detect changes efficiently.

---

## Core Ideas

### 1. **State = mutable data owned by a component**

Unlike props (passed down, read-only), state is:
- **internal** to a component
- **mutable** (can change over time)
- **triggers re-render** when updated

**Mental model:**  
"State is component memory that causes re-renders when changed."

---

### 2. **useState creates state variables**

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);  // Initial value: 0
  //      ^        ^            ^
  //    current  updater      initial
  //    value    function      value
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
}
```

**Key:** Always use the updater function; never mutate state directly.

---

### 3. **State updates trigger re-renders**

When you call `setState`:
1. React schedules a re-render
2. Component function runs again
3. New JSX is compared with previous (Virtual DOM diffing)
4. Minimal DOM updates are applied

**Important:** State updates are **asynchronous** and **batched**.

---

### 4. **Multiple state variables**

```jsx
function UserProfile() {
  const [name, setName] = useState('Alice');
  const [age, setAge] = useState(25);
  const [email, setEmail] = useState('alice@example.com');
  
  // Each piece of state can be updated independently
}
```

**Or use object state:**
```jsx
function UserProfile() {
  const [user, setUser] = useState({
    name: 'Alice',
    age: 25,
    email: 'alice@example.com'
  });
  
  // Update: must spread previous state
  setUser({ ...user, age: 26 });
}
```

---

### 5. **Lifecycle in function components**

Function components don't have lifecycle methods (that's class components).

**Instead, use hooks:**

| Need | Hook |
|------|------|
| After render | `useEffect(() => {...}, [])` |
| Before unmount | `useEffect(() => { return () => {...} }, [])` |
| On state change | `useEffect(() => {...}, [state])` |

---

## Examples

### Example 1 — Simple counter

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

---

### Example 2 — Form input (controlled component)

```jsx
function NameForm() {
  const [name, setName] = useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault();
    alert(`Hello, ${name}!`);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input 
        type="text"
        value={name}
        onChange={(e) => setName(e.target.value)}
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

**Key:** `value={name}` makes React control the input.

---

### Example 3 — Object state updates

```jsx
function UserEditor() {
  const [user, setUser] = useState({
    name: 'Alice',
    email: 'alice@example.com'
  });
  
  const updateName = (newName) => {
    // ❌ Wrong: mutates state
    // user.name = newName;
    
    // ✅ Right: create new object
    setUser({ ...user, name: newName });
    
    // Also correct: functional update
    setUser(prevUser => ({ ...prevUser, name: newName }));
  };
  
  return (
    <div>
      <input 
        value={user.name}
        onChange={(e) => updateName(e.target.value)}
      />
    </div>
  );
}
```

---

### Example 4 — Functional state updates (when new state depends on old)

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  
  const incrementTwice = () => {
    // ❌ Won't work as expected (batching)
    setCount(count + 1);
    setCount(count + 1);  // Still uses old count
    
    // ✅ Functional update (uses previous value)
    setCount(c => c + 1);
    setCount(c => c + 1);  // Now works correctly
  };
  
  return <button onClick={incrementTwice}>+2</button>;
}
```

---

### Example 5 — Lazy initialization

```jsx
function ExpensiveComponent() {
  // ❌ Runs on every render
  const [data, setData] = useState(expensiveCalculation());
  
  // ✅ Runs only once (initial render)
  const [data, setData] = useState(() => expensiveCalculation());
  
  return <div>{data}</div>;
}
```

Use function initializer for expensive computations.

---

## Common Pitfalls

### 1. **Mutating state directly**

```jsx
// ❌ Never mutate state!
function BadCounter() {
  const [count, setCount] = useState(0);
  
  const increment = () => {
    count++;  // Doesn't trigger re-render!
  };
}

// ✅ Always use setter
function GoodCounter() {
  const [count, setCount] = useState(0);
  
  const increment = () => {
    setCount(count + 1);  // Triggers re-render
  };
}
```

---

### 2. **Not spreading object/array state**

```jsx
// ❌ Mutates existing object
const updateUser = () => {
  user.name = 'Bob';
  setUser(user);  // React may not detect change
};

// ✅ Create new object
const updateUser = () => {
  setUser({ ...user, name: 'Bob' });
};

// ❌ Mutates array
const addItem = () => {
  items.push(newItem);
  setItems(items);
};

// ✅ Create new array
const addItem = () => {
  setItems([...items, newItem]);
};
```

---

### 3. **Using stale state in callbacks**

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      // ❌ Captures initial count (0) forever
      setCount(count + 1);
    }, 1000);
    
    return () => clearInterval(interval);
  }, []);  // Empty deps = stale closure
  
  // ✅ Use functional update
  useEffect(() => {
    const interval = setInterval(() => {
      setCount(c => c + 1);  // Always uses current value
    }, 1000);
    
    return () => clearInterval(interval);
  }, []);
}
```

---

### 4. **Setting state during render**

```jsx
// ❌ Infinite loop!
function Bad() {
  const [count, setCount] = useState(0);
  setCount(count + 1);  // Called during render → triggers render → infinite
  return <div>{count}</div>;
}

// ✅ Set state in event handlers or effects
function Good() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    setCount(1);  // Runs after render
  }, []);
  
  return <div>{count}</div>;
}
```

---

### 5. **Comparing props and state incorrectly**

| Feature | Props | State |
|---------|-------|-------|
| Owned by | Parent | Component itself |
| Mutable? | No | Yes (via setState) |
| Triggers re-render? | When parent re-renders | When setState called |

---

## FAQ

**Q: When should I use state vs props?**  
A: Props = data from parent. State = data component manages itself.

**Q: Can I use state in every component?**  
A: Yes, but lift state up to common ancestor when multiple components need it.

**Q: Why is state update asynchronous?**  
A: For performance. React batches multiple updates into one re-render.

**Q: How do I update state based on previous state?**  
A: Use functional update: `setState(prev => prev + 1)`

**Q: What's the difference between class and function component state?**  
A: Class: `this.state` + `this.setState`. Function: `useState` hook.

**Q: Can I call useState conditionally?**  
A: **No!** Hooks must be called in the same order every render.

---

## Quick Self-Check

1. What's wrong with this code?
   ```jsx
   const [count, setCount] = useState(0);
   count = count + 1;  // ❌
   ```

2. How do you update object state correctly?
   ```jsx
   setUser({ ...user, name: 'Bob' });
   ```

3. When does React re-render a component?
   - When state changes
   - When props change
   - When parent re-renders

4. What's the difference between these?
   ```jsx
   setCount(count + 1);          // Uses current value
   setCount(c => c + 1);         // Uses previous value
   ```

5. Why use functional updates?
   - Avoids stale closures
   - Safe for async updates
   - Guarantees latest value

---

## Further Practice

1. **Build:** Todo list (add, remove, toggle items)
2. **Build:** Form with multiple inputs (name, email, message)
3. **Experiment:** Create state loops and debug them
4. **Optimize:** Convert class component state to hooks
5. **Debug:** Fix stale closure bugs in setInterval/setTimeout

---

## Key Takeaways

- State is component-owned, mutable data that triggers re-renders
- Use `useState` hook: `const [value, setValue] = useState(initial)`
- Never mutate state directly—always use setter function
- State updates are asynchronous and batched
- Use functional updates when new state depends on old: `setState(prev => ...)`
- Spread objects/arrays when updating nested state
- Props flow down; state is local to component
