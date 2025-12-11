# Refs & DOM Access

**Level:** 🚦 Intermediate  
**Tags:** `refs`, `useRef`, `forwardRef`, `useImperativeHandle`, `DOM`, `interview`

---

## Why this matters

Refs provide direct access to DOM nodes and persistent values that survive re-renders without causing updates. While React is declarative, certain scenarios require imperative operations—focus management, animations, third-party library integration, and measuring elements. Interviewers ask about refs to assess when you understand the trade-offs between declarative and imperative code, and whether you know React's escape hatches for DOM manipulation.

**Why interviewers care:**
- Refs show you understand React's rendering model and when to bypass it
- forwardRef questions assess component API design knowledge
- Knowing when NOT to use refs demonstrates React best practices
- useImperativeHandle tests advanced ref patterns and encapsulation
- Senior roles require designing accessible components (focus management)
- Refs are essential for integrating non-React libraries (charts, maps)

**Real-world implications:**
- **Focus management:** Auto-focus inputs, trap focus in modals, keyboard navigation
- **Animations:** Trigger imperative animations (GSAP, anime.js integration)
- **Measurements:** Get element dimensions for responsive layouts
- **Third-party integration:** Attach libraries to DOM nodes (video players, editors)
- **Scroll position:** Programmatically scroll elements
- **Persistent values:** Store mutable values without triggering re-renders
- **Accessibility:** Manage focus for screen readers and keyboard users

**Common scenarios requiring refs:**
- Managing focus (auto-focus forms, focus traps in modals)
- Measuring element size/position (responsive components, tooltips)
- Integrating non-React libraries (D3, Three.js, video players)
- Triggering animations imperatively
- Storing previous values (comparing old vs new props/state)
- Storing timers/intervals that shouldn't trigger renders
- Scrolling elements programmatically

**What you must know:**
- **useRef:** Creates mutable ref object with `.current` property
- **Refs don't trigger re-renders:** Changing `.current` doesn't cause updates
- **DOM refs:** Access underlying DOM nodes
- **forwardRef:** Pass refs through components
- **useImperativeHandle:** Customize ref value exposed to parent
- **When to avoid refs:** Don't use for things React can handle declaratively
- **Ref timing:** Refs available after component mounts
- **Callback refs:** Functions called with DOM node

**Interview red flag:** Overusing refs for things that should be state (anti-pattern), or not knowing forwardRef when building reusable components shows lack of production experience. Using refs instead of controlled components for form inputs is a major mistake.

---

## Core Ideas

### 1. **useRef creates persistent mutable reference**

```jsx
function Component() {
  const countRef = useRef(0);
  const inputRef = useRef(null);
  
  const handleClick = () => {
    countRef.current += 1;  // Doesn't trigger re-render!
    console.log('Clicked:', countRef.current);
  };
  
  return (
    <div>
      <input ref={inputRef} />
      <button onClick={handleClick}>Click</button>
    </div>
  );
}
```

**Mental model:**  
"useRef is like state that doesn't trigger re-renders when changed."

---

### 2. **Refs provide direct DOM access**

```jsx
function FocusInput() {
  const inputRef = useRef(null);
  
  const focusInput = () => {
    inputRef.current.focus();  // Direct DOM manipulation
  };
  
  return (
    <>
      <input ref={inputRef} />
      <button onClick={focusInput}>Focus Input</button>
    </>
  );
}
```

**Key:** Attach ref to element with `ref` prop, access node via `.current`.

---

### 3. **forwardRef passes refs through components**

```jsx
// Without forwardRef: Can't access inner input
function Input(props) {
  return <input {...props} />;
}

// With forwardRef: Parent can access inner input
const Input = forwardRef((props, ref) => {
  return <input ref={ref} {...props} />;
});

// Usage
function Parent() {
  const inputRef = useRef(null);
  
  return <Input ref={inputRef} />;
}
```

---

### 4. **useImperativeHandle customizes exposed ref value**

```jsx
const Input = forwardRef((props, ref) => {
  const inputRef = useRef(null);
  
  // Expose only focus method, not entire DOM node
  useImperativeHandle(ref, () => ({
    focus: () => inputRef.current.focus(),
    scrollIntoView: () => inputRef.current.scrollIntoView()
  }));
  
  return <input ref={inputRef} {...props} />;
});

// Parent can only call exposed methods
function Parent() {
  const inputRef = useRef(null);
  
  return (
    <>
      <Input ref={inputRef} />
      <button onClick={() => inputRef.current.focus()}>Focus</button>
    </>
  );
}
```

---

### 5. **Refs persist across renders**

```jsx
function Timer() {
  const [count, setCount] = useState(0);
  const intervalRef = useRef(null);
  
  useEffect(() => {
    // Store interval ID in ref (persists across renders)
    intervalRef.current = setInterval(() => {
      setCount(c => c + 1);
    }, 1000);
    
    return () => clearInterval(intervalRef.current);
  }, []);
  
  return <div>{count}</div>;
}
```

---

## Examples

### Example 1 — Focus management

```jsx
function LoginForm() {
  const usernameRef = useRef(null);
  const passwordRef = useRef(null);
  
  // Auto-focus username on mount
  useEffect(() => {
    usernameRef.current?.focus();
  }, []);
  
  const handleSubmit = (e) => {
    e.preventDefault();
    
    // Validate and focus error field
    if (!usernameRef.current.value) {
      usernameRef.current.focus();
      return;
    }
    
    if (!passwordRef.current.value) {
      passwordRef.current.focus();
      return;
    }
    
    // Submit form
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input ref={usernameRef} placeholder="Username" />
      <input ref={passwordRef} type="password" placeholder="Password" />
      <button type="submit">Login</button>
    </form>
  );
}
```

---

### Example 2 — Measuring element dimensions

```jsx
function ResponsiveComponent() {
  const divRef = useRef(null);
  const [width, setWidth] = useState(0);
  
  useEffect(() => {
    // Measure element on mount
    const updateWidth = () => {
      if (divRef.current) {
        setWidth(divRef.current.offsetWidth);
      }
    };
    
    updateWidth();
    
    // Re-measure on resize
    window.addEventListener('resize', updateWidth);
    return () => window.removeEventListener('resize', updateWidth);
  }, []);
  
  return (
    <div ref={divRef}>
      Width: {width}px
    </div>
  );
}
```

---

### Example 3 — forwardRef for reusable components

```jsx
// Reusable Input component with ref support
const Input = forwardRef(({ label, error, ...props }, ref) => {
  return (
    <div className="input-wrapper">
      <label>{label}</label>
      <input 
        ref={ref} 
        className={error ? 'error' : ''}
        {...props} 
      />
      {error && <span className="error-message">{error}</span>}
    </div>
  );
});

// Usage
function Form() {
  const emailRef = useRef(null);
  const passwordRef = useRef(null);
  const [errors, setErrors] = useState({});
  
  const handleSubmit = (e) => {
    e.preventDefault();
    
    const newErrors = {};
    
    if (!emailRef.current.value) {
      newErrors.email = 'Email is required';
      emailRef.current.focus();
    }
    
    if (!passwordRef.current.value) {
      newErrors.password = 'Password is required';
      if (!newErrors.email) passwordRef.current.focus();
    }
    
    setErrors(newErrors);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <Input 
        ref={emailRef}
        label="Email"
        error={errors.email}
      />
      <Input 
        ref={passwordRef}
        type="password"
        label="Password"
        error={errors.password}
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

### Example 4 — useImperativeHandle for custom API

```jsx
const VideoPlayer = forwardRef((props, ref) => {
  const videoRef = useRef(null);
  const [isPlaying, setIsPlaying] = useState(false);
  
  // Expose custom API instead of raw DOM node
  useImperativeHandle(ref, () => ({
    play: () => {
      videoRef.current?.play();
      setIsPlaying(true);
    },
    pause: () => {
      videoRef.current?.pause();
      setIsPlaying(false);
    },
    seek: (time) => {
      if (videoRef.current) {
        videoRef.current.currentTime = time;
      }
    },
    getCurrentTime: () => videoRef.current?.currentTime || 0,
    getDuration: () => videoRef.current?.duration || 0
  }));
  
  return (
    <div>
      <video ref={videoRef} src={props.src} />
      <p>{isPlaying ? 'Playing' : 'Paused'}</p>
    </div>
  );
});

// Parent controls video through custom API
function VideoControls() {
  const playerRef = useRef(null);
  
  return (
    <div>
      <VideoPlayer ref={playerRef} src="/video.mp4" />
      
      <button onClick={() => playerRef.current.play()}>Play</button>
      <button onClick={() => playerRef.current.pause()}>Pause</button>
      <button onClick={() => playerRef.current.seek(30)}>
        Skip to 30s
      </button>
    </div>
  );
}
```

---

### Example 5 — Storing previous value

```jsx
function usePrevious(value) {
  const ref = useRef();
  
  useEffect(() => {
    ref.current = value;  // Update ref after render
  });
  
  return ref.current;  // Return previous value
}

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

### Example 6 — Integrating third-party library

```jsx
function ChartComponent({ data }) {
  const chartRef = useRef(null);
  const chartInstanceRef = useRef(null);
  
  useEffect(() => {
    // Initialize chart library on mount
    if (chartRef.current && !chartInstanceRef.current) {
      chartInstanceRef.current = new Chart(chartRef.current, {
        type: 'bar',
        data: data
      });
    }
    
    // Update chart when data changes
    if (chartInstanceRef.current) {
      chartInstanceRef.current.data = data;
      chartInstanceRef.current.update();
    }
    
    // Cleanup on unmount
    return () => {
      chartInstanceRef.current?.destroy();
    };
  }, [data]);
  
  return <canvas ref={chartRef} />;
}
```

---

### Example 7 — Callback refs for dynamic DOM access

```jsx
function MeasureHeight() {
  const [height, setHeight] = useState(0);
  
  // Callback ref: Called with DOM node
  const measuredRef = useCallback(node => {
    if (node !== null) {
      setHeight(node.getBoundingClientRect().height);
    }
  }, []);
  
  return (
    <div>
      <div ref={measuredRef}>
        <p>Content with dynamic height</p>
      </div>
      <p>Height: {height}px</p>
    </div>
  );
}
```

---

## Common Pitfalls

### 1. **Using refs instead of state**

```jsx
// ❌ Ref changes don't trigger re-renders
function Counter() {
  const countRef = useRef(0);
  
  const increment = () => {
    countRef.current += 1;
    // UI won't update!
  };
  
  return (
    <div>
      {countRef.current}
      <button onClick={increment}>+</button>
    </div>
  );
}

// ✅ Use state for values that affect rendering
function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      {count}
      <button onClick={() => setCount(c => c + 1)}>+</button>
    </div>
  );
}
```

---

### 2. **Accessing ref.current during render**

```jsx
// ❌ Ref not guaranteed to be set during render
function Component() {
  const ref = useRef(null);
  
  // Bad: ref.current might be null
  const width = ref.current?.offsetWidth || 0;
  
  return <div ref={ref}>Width: {width}</div>;
}

// ✅ Access refs in useEffect (after DOM exists)
function Component() {
  const ref = useRef(null);
  const [width, setWidth] = useState(0);
  
  useEffect(() => {
    if (ref.current) {
      setWidth(ref.current.offsetWidth);
    }
  }, []);
  
  return <div ref={ref}>Width: {width}</div>;
}
```

---

### 3. **Forgetting forwardRef for reusable components**

```jsx
// ❌ Ref not forwarded to DOM element
function Input(props) {
  return <input {...props} />;
}

function Parent() {
  const ref = useRef(null);
  return <Input ref={ref} />;  // Warning: ref not forwarded
}

// ✅ Use forwardRef
const Input = forwardRef((props, ref) => {
  return <input ref={ref} {...props} />;
});
```

---

### 4. **Using refs for uncontrolled components when controlled is better**

```jsx
// ❌ Uncontrolled (hard to validate, test, sync with other state)
function Form() {
  const inputRef = useRef(null);
  
  const handleSubmit = () => {
    console.log(inputRef.current.value);
  };
  
  return <input ref={inputRef} />;
}

// ✅ Controlled component (React manages value)
function Form() {
  const [value, setValue] = useState('');
  
  const handleSubmit = () => {
    console.log(value);
  };
  
  return <input value={value} onChange={e => setValue(e.target.value)} />;
}
```

---

### 5. **Not cleaning up refs in useEffect**

```jsx
// ❌ Interval not cleared
function Timer() {
  const intervalRef = useRef(null);
  
  useEffect(() => {
    intervalRef.current = setInterval(() => {
      console.log('tick');
    }, 1000);
    // Missing cleanup!
  }, []);
}

// ✅ Clear interval in cleanup
function Timer() {
  const intervalRef = useRef(null);
  
  useEffect(() => {
    intervalRef.current = setInterval(() => {
      console.log('tick');
    }, 1000);
    
    return () => clearInterval(intervalRef.current);
  }, []);
}
```

---

## FAQ

**Q: When should I use refs vs state?**  
A: State for values that affect rendering. Refs for values that don't (timers, DOM nodes, previous values).

**Q: Why doesn't changing ref.current trigger re-render?**  
A: By design. Refs are for mutable values that shouldn't cause updates.

**Q: When should I use forwardRef?**  
A: When building reusable components that need to expose underlying DOM nodes to parents.

**Q: What's useImperativeHandle for?**  
A: Customizing the ref value exposed to parent (hide implementation, expose only specific methods).

**Q: Can I use refs in function components?**  
A: Yes, with `useRef` hook.

**Q: What are callback refs?**  
A: Functions passed to `ref` prop, called with DOM node (useful for dynamic refs).

**Q: Should I use refs for form inputs?**  
A: Prefer controlled components (state). Use refs only for focus management or third-party integration.

---

## Quick Self-Check

1. What's wrong with this?
   ```jsx
   const ref = useRef(0);
   return <div>{ref.current}</div>;
   ```
   (Changing `ref.current` won't trigger re-render; use state instead)

2. How do you pass refs through custom components?
   (Use `forwardRef`)

3. When is ref.current guaranteed to be set?
   (After component mounts, in `useEffect`)

4. What's useImperativeHandle used for?
   (Customizing the ref value exposed to parent)

5. Should you use refs for form input values?
   (No, use controlled components with state unless specific reason)

---

## Further Practice

1. **Build:** Auto-focus first input in form with validation focus on errors
2. **Create:** Custom Input component with forwardRef
3. **Implement:** useImperativeHandle to expose only specific methods
4. **Build:** Element dimension tracker (resize observer with refs)
5. **Integrate:** Third-party library (chart, video player) with refs
6. **Create:** usePrevious custom hook with refs
7. **Build:** Modal with focus trap using refs

---

## Key Takeaways

- useRef creates mutable ref object with `.current` property
- Refs don't trigger re-renders when `.current` changes
- Use refs for DOM access, timers, previous values, third-party libraries
- Don't use refs for values that affect rendering (use state)
- forwardRef passes refs through custom components to underlying DOM
- useImperativeHandle customizes ref value exposed to parent
- Access refs in useEffect (after mount), not during render
- Callback refs are functions called with DOM node
- Prefer controlled components over refs for form inputs
- Clean up refs in useEffect cleanup (intervals, listeners)
