# Event Handling

**Level:** 🏁 Beginner  
**Tags:** `events`, `synthetic-events`, `event-binding`, `interview`, `fundamentals`

---

## Why this matters

Event handling is how users interact with your app. In interviews, you must:

- understand React's **Synthetic Event** system
- know how to handle events without binding issues
- prevent common event bugs (memory leaks, performance issues)
- explain event delegation

**Interview red flag:** binding `this` in function components or creating functions in render

---

## Core Ideas

### 1. **React uses Synthetic Events (cross-browser wrapper)**

React wraps native browser events in a `SyntheticEvent` object:

```jsx
function handleClick(event) {
  console.log(event);  // SyntheticEvent, not native Event
  console.log(event.nativeEvent);  // Access native event if needed
}
```

**Why?**
- Cross-browser consistency
- Performance optimizations (event pooling in React < 17)
- Additional features

**Mental model:**  
"SyntheticEvent = React's normalized version of browser events."

---

### 2. **Event naming is camelCase**

| HTML | React |
|------|-------|
| `onclick` | `onClick` |
| `onchange` | `onChange` |
| `onsubmit` | `onSubmit` |
| `onkeypress` | `onKeyPress` |

```jsx
// ❌ HTML style
<button onclick="handleClick()">Click</button>

// ✅ React style
<button onClick={handleClick}>Click</button>
```

---

### 3. **Pass function reference, not call**

```jsx
// ❌ Calls function immediately on render
<button onClick={handleClick()}>Click</button>

// ✅ Passes function reference
<button onClick={handleClick}>Click</button>

// ✅ Inline arrow function (when you need to pass args)
<button onClick={() => handleClick(id)}>Click</button>
```

---

### 4. **Prevent default and stop propagation**

```jsx
function handleSubmit(event) {
  event.preventDefault();  // Stop form submission
  event.stopPropagation(); // Stop event bubbling
  
  // Your logic here
}

<form onSubmit={handleSubmit}>...</form>
```

**Note:** Can't use `return false` to prevent default like in HTML.

---

### 5. **Event delegation (React handles it automatically)**

React uses event delegation at the root level:
- Attaches one listener to the root
- Uses event bubbling to handle all events
- More efficient than attaching listeners to every element

**You don't need to worry about this—it's automatic.**

---

## Examples

### Example 1 — Basic click handler

```jsx
function Button() {
  const handleClick = () => {
    console.log('Button clicked!');
  };
  
  return <button onClick={handleClick}>Click me</button>;
}
```

---

### Example 2 — Passing arguments to handlers

```jsx
function ItemList({ items }) {
  const handleDelete = (id) => {
    console.log(`Delete item ${id}`);
  };
  
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>
          {item.name}
          {/* ❌ Calls immediately */}
          <button onClick={handleDelete(item.id)}>Delete</button>
          
          {/* ✅ Passes function */}
          <button onClick={() => handleDelete(item.id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
}
```

---

### Example 3 — Form events

```jsx
function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault();  // Prevent page reload
    console.log('Login:', email, password);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
      />
      <button type="submit">Login</button>
    </form>
  );
}
```

---

### Example 4 — Keyboard events

```jsx
function SearchBox() {
  const handleKeyDown = (e) => {
    if (e.key === 'Enter') {
      console.log('Enter pressed!');
    }
    if (e.key === 'Escape') {
      console.log('Escape pressed!');
    }
  };
  
  return (
    <input
      type="text"
      placeholder="Search..."
      onKeyDown={handleKeyDown}
    />
  );
}
```

---

### Example 5 — Mouse events

```jsx
function HoverCard() {
  const [isHovered, setIsHovered] = useState(false);
  
  return (
    <div
      onMouseEnter={() => setIsHovered(true)}
      onMouseLeave={() => setIsHovered(false)}
      style={{
        background: isHovered ? 'lightblue' : 'white',
        padding: '20px'
      }}
    >
      {isHovered ? 'Hovering!' : 'Hover over me'}
    </div>
  );
}
```

---

## Common Pitfalls

### 1. **Calling function instead of passing reference**

```jsx
// ❌ Calls on every render
<button onClick={handleClick()}>Click</button>

// ✅ Passes reference
<button onClick={handleClick}>Click</button>

// ✅ Use arrow function if you need args
<button onClick={() => handleClick(id)}>Click</button>
```

---

### 2. **Creating functions in render (performance)**

```jsx
// ❌ Creates new function on every render
function List({ items }) {
  return items.map(item => (
    <button onClick={() => console.log(item.id)}>
      {item.name}
    </button>
  ));
}

// ✅ Extract to component (React.memo can optimize)
function ListItem({ item }) {
  const handleClick = () => console.log(item.id);
  return <button onClick={handleClick}>{item.name}</button>;
}

// ✅ Or use useCallback for expensive handlers
function List({ items }) {
  const handleClick = useCallback((id) => {
    console.log(id);
  }, []);
  
  return items.map(item => (
    <button onClick={() => handleClick(item.id)}>
      {item.name}
    </button>
  ));
}
```

---

### 3. **Accessing event asynchronously**

```jsx
// ❌ SyntheticEvent is pooled (React < 17)
function Bad() {
  const handleClick = (e) => {
    setTimeout(() => {
      console.log(e.target);  // May be null!
    }, 1000);
  };
}

// ✅ Extract values immediately
function Good() {
  const handleClick = (e) => {
    const value = e.target.value;  // Capture now
    setTimeout(() => {
      console.log(value);  // Safe to use
    }, 1000);
  };
}

// Note: React 17+ doesn't pool events, but it's still good practice
```

---

### 4. **Not preventing default for forms/links**

```jsx
// ❌ Page reloads on submit
<form onSubmit={handleSubmit}>

// ✅ Prevent default
function handleSubmit(e) {
  e.preventDefault();
  // Handle form
}
```

---

### 5. **Binding `this` in class components (legacy)**

```jsx
// Legacy class component issue (not relevant for function components)

class Button extends React.Component {
  // ❌ `this` is undefined
  handleClick() {
    console.log(this.state);
  }
  
  // ✅ Bind in constructor
  constructor(props) {
    super(props);
    this.handleClick = this.handleClick.bind(this);
  }
  
  // ✅ Or use arrow function
  handleClick = () => {
    console.log(this.state);
  }
}

// ✅ Function components don't have this problem
function Button() {
  const handleClick = () => { ... };
}
```

---

## FAQ

**Q: What's the difference between SyntheticEvent and native Event?**  
A: SyntheticEvent wraps native events for cross-browser consistency and React optimizations.

**Q: Can I access the native event?**  
A: Yes, use `event.nativeEvent`.

**Q: Why `onClick` instead of `onclick`?**  
A: JSX uses camelCase. Also, you pass functions not strings.

**Q: Does React attach listeners to every element?**  
A: No. React uses event delegation—one listener at the root.

**Q: When should I use `useCallback` for event handlers?**  
A: When passing handlers to memoized child components or when the handler is a dependency.

**Q: Can I use `return false` to prevent default?**  
A: No. Use `event.preventDefault()`.

---

## Quick Self-Check

1. What's wrong with this?
   ```jsx
   <button onClick={handleClick()}>Click</button>
   ```
   (Calls function immediately, not on click)

2. How do you prevent form submission?
   ```jsx
   const handleSubmit = (e) => {
     e.preventDefault();
   };
   ```

3. What's the difference?
   ```jsx
   onClick={handleClick}           // Pass reference
   onClick={() => handleClick()}   // Call in arrow function
   ```

4. What does React's event delegation mean?
   (One listener at root, uses bubbling for efficiency)

5. When do you need to extract event properties?
   (When accessing them asynchronously, like in setTimeout)

---

## Further Practice

1. **Build:** Form with multiple inputs (onChange handlers)
2. **Build:** Modal that closes on Escape key
3. **Build:** Drag-and-drop interface (mouse events)
4. **Optimize:** Refactor inline handlers to useCallback
5. **Debug:** Create and fix event handler performance issues

---

## Key Takeaways

- React uses SyntheticEvent for cross-browser consistency
- Event names are camelCase: `onClick`, `onChange`, etc.
- Pass function reference, not call: `onClick={fn}` not `onClick={fn()}`
- Use `e.preventDefault()` to prevent default behavior
- Use `e.stopPropagation()` to stop event bubbling
- Extract event values before async operations
- React handles event delegation automatically
- Avoid creating functions in render for performance
- Function components don't have `this` binding issues
