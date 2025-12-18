# Advanced Hooks

**Level:** 🔥 Advanced  
**Tags:** `hooks`, `useReducer`, `useImperativeHandle`, `useLayoutEffect`, `useDebugValue`, `interview`

---

## Why this matters

While useState and useEffect cover 80% of use cases, advanced hooks solve specific problems that come up in real production applications. Interviewers use advanced hook questions to assess whether you understand React deeply and can choose the right tool for complex scenarios. These hooks separate developers who memorize patterns from those who understand React's architecture.

**Why interviewers care:**
- Advanced hooks reveal depth of React knowledge beyond basics
- useReducer questions test state management understanding and when to scale beyond useState
- useImperativeHandle tests understanding of React's ref system and when to break encapsulation
- useLayoutEffect tests understanding of rendering lifecycle and browser paint cycle
- Hook selection (useState vs. useReducer) shows architectural decision-making skills
- Custom hook composition with advanced hooks demonstrates mastery

**Real-world implications:**
- **useReducer:** Managing complex state logic (multi-step forms, shopping carts, game state)
- **useImperativeHandle:** Third-party library integration, focus management, imperative animations
- **useLayoutEffect:** Measuring DOM, preventing visual flicker, tooltip positioning
- **useDebugValue:** Debugging custom hooks in React DevTools
- **Complex apps:** Advanced hooks are essential for large-scale applications

**Common scenarios requiring advanced hooks:**
- State with complex update logic (useReducer)
- Exposing component methods to parent (useImperativeHandle)
- Reading layout before paint (useLayoutEffect)
- Form wizards with multiple steps (useReducer)
- Coordinating multiple related state values (useReducer)
- Preventing flash of incorrect content (useLayoutEffect)
- Building reusable component libraries (useImperativeHandle)

**What you must know:**
- **useReducer:** When to use vs. useState, reducer pattern, dispatch
- **useImperativeHandle:** Ref forwarding, exposing imperative handles
- **useLayoutEffect:** Difference from useEffect, synchronous timing
- **useDebugValue:** Displaying custom hook values in DevTools
- **useMemo / useCallback:** Memoization for performance (covered in optimization lesson)

**Interview red flags:** Not knowing when useReducer is better than useState, or using useLayoutEffect when useEffect would work (premature optimization).

---

## Core Ideas

### 1. **useReducer: Complex State Logic**

**When to use useReducer instead of useState:**
- State has complex update logic (multiple related values)
- Next state depends on previous state in non-trivial ways
- You're managing state with many different actions
- You want to centralize state update logic
- You're mimicking Redux pattern locally

**Basic example:**

```jsx
import { useReducer } from 'react';

// State type
interface State {
  count: number;
}

// Action type
type Action = 
  | { type: 'increment' }
  | { type: 'decrement' }
  | { type: 'reset' }
  | { type: 'set'; payload: number };

// Reducer: (state, action) => newState
function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return { count: 0 };
    case 'set':
      return { count: action.payload };
    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
      <button onClick={() => dispatch({ type: 'set', payload: 10 })}>Set to 10</button>
    </div>
  );
}
```

**Real-world example: Shopping cart**

```jsx
interface CartItem {
  id: string;
  name: string;
  price: number;
  quantity: number;
}

interface CartState {
  items: CartItem[];
  total: number;
}

type CartAction =
  | { type: 'ADD_ITEM'; payload: CartItem }
  | { type: 'REMOVE_ITEM'; payload: string }  // item id
  | { type: 'UPDATE_QUANTITY'; payload: { id: string; quantity: number } }
  | { type: 'CLEAR_CART' };

function cartReducer(state: CartState, action: CartAction): CartState {
  switch (action.type) {
    case 'ADD_ITEM': {
      const existingItem = state.items.find(item => item.id === action.payload.id);
      
      if (existingItem) {
        // Item exists, increase quantity
        const updatedItems = state.items.map(item =>
          item.id === action.payload.id
            ? { ...item, quantity: item.quantity + 1 }
            : item
        );
        return {
          items: updatedItems,
          total: calculateTotal(updatedItems)
        };
      }
      
      // New item
      const newItems = [...state.items, action.payload];
      return {
        items: newItems,
        total: calculateTotal(newItems)
      };
    }
    
    case 'REMOVE_ITEM': {
      const filtered = state.items.filter(item => item.id !== action.payload);
      return {
        items: filtered,
        total: calculateTotal(filtered)
      };
    }
    
    case 'UPDATE_QUANTITY': {
      const updated = state.items.map(item =>
        item.id === action.payload.id
          ? { ...item, quantity: action.payload.quantity }
          : item
      );
      return {
        items: updated,
        total: calculateTotal(updated)
      };
    }
    
    case 'CLEAR_CART':
      return { items: [], total: 0 };
    
    default:
      return state;
  }
}

function calculateTotal(items: CartItem[]): number {
  return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
}

function ShoppingCart() {
  const [state, dispatch] = useReducer(cartReducer, { items: [], total: 0 });

  const addItem = (item: CartItem) => {
    dispatch({ type: 'ADD_ITEM', payload: item });
  };

  const removeItem = (id: string) => {
    dispatch({ type: 'REMOVE_ITEM', payload: id });
  };

  return (
    <div>
      <h2>Shopping Cart</h2>
      {state.items.map(item => (
        <div key={item.id}>
          <span>{item.name} - ${item.price} x {item.quantity}</span>
          <button onClick={() => removeItem(item.id)}>Remove</button>
        </div>
      ))}
      <p>Total: ${state.total}</p>
      <button onClick={() => dispatch({ type: 'CLEAR_CART' })}>Clear Cart</button>
    </div>
  );
}
```

**useReducer with lazy initialization:**

```jsx
function init(initialCount: number) {
  // Expensive computation
  return { count: initialCount * 2 };
}

function Counter({ initialCount }: { initialCount: number }) {
  // Third argument is init function (runs only once)
  const [state, dispatch] = useReducer(reducer, initialCount, init);
  
  return <div>{state.count}</div>;
}
```

**When useReducer is better than useState:**

```jsx
// ❌ BAD: Multiple related useState calls
function Form() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [errors, setErrors] = useState({});
  
  // Complex update logic scattered across component
  const handleSubmit = async () => {
    setIsSubmitting(true);
    setErrors({});
    // ... validation logic
    setIsSubmitting(false);
  };
}

// ✅ GOOD: Centralized with useReducer
interface FormState {
  fields: { name: string; email: string };
  isSubmitting: boolean;
  errors: Record<string, string>;
}

type FormAction =
  | { type: 'UPDATE_FIELD'; field: string; value: string }
  | { type: 'SUBMIT_START' }
  | { type: 'SUBMIT_SUCCESS' }
  | { type: 'SUBMIT_ERROR'; errors: Record<string, string> };

function formReducer(state: FormState, action: FormAction): FormState {
  switch (action.type) {
    case 'UPDATE_FIELD':
      return {
        ...state,
        fields: { ...state.fields, [action.field]: action.value }
      };
    case 'SUBMIT_START':
      return { ...state, isSubmitting: true, errors: {} };
    case 'SUBMIT_SUCCESS':
      return { ...state, isSubmitting: false, fields: { name: '', email: '' } };
    case 'SUBMIT_ERROR':
      return { ...state, isSubmitting: false, errors: action.errors };
    default:
      return state;
  }
}

function Form() {
  const [state, dispatch] = useReducer(formReducer, {
    fields: { name: '', email: '' },
    isSubmitting: false,
    errors: {}
  });
  
  // Cleaner update logic
  const handleSubmit = async () => {
    dispatch({ type: 'SUBMIT_START' });
    
    try {
      await submitForm(state.fields);
      dispatch({ type: 'SUBMIT_SUCCESS' });
    } catch (errors) {
      dispatch({ type: 'SUBMIT_ERROR', errors });
    }
  };
}
```

---

### 2. **useImperativeHandle: Exposing Component Methods**

**Problem:** Sometimes you need to expose component methods to parent (imperative API).

**useImperativeHandle with forwardRef:**

```jsx
import { useRef, useImperativeHandle, forwardRef } from 'react';

// Define the handle type
interface InputHandle {
  focus: () => void;
  clear: () => void;
  getValue: () => string;
}

// Component that exposes methods
const FancyInput = forwardRef<InputHandle, { placeholder?: string }>(
  (props, ref) => {
    const inputRef = useRef<HTMLInputElement>(null);

    // Expose custom methods to parent
    useImperativeHandle(ref, () => ({
      focus: () => {
        inputRef.current?.focus();
      },
      clear: () => {
        if (inputRef.current) {
          inputRef.current.value = '';
        }
      },
      getValue: () => {
        return inputRef.current?.value || '';
      }
    }));

    return <input ref={inputRef} placeholder={props.placeholder} />;
  }
);

// Parent component
function Form() {
  const inputRef = useRef<InputHandle>(null);

  const handleSubmit = () => {
    const value = inputRef.current?.getValue();
    console.log('Value:', value);
    inputRef.current?.clear();
    inputRef.current?.focus();
  };

  return (
    <div>
      <FancyInput ref={inputRef} placeholder="Enter text" />
      <button onClick={handleSubmit}>Submit</button>
    </div>
  );
}
```

**Real-world example: Video player**

```jsx
interface VideoPlayerHandle {
  play: () => void;
  pause: () => void;
  seek: (time: number) => void;
  getCurrentTime: () => number;
}

const VideoPlayer = forwardRef<VideoPlayerHandle, { src: string }>(
  ({ src }, ref) => {
    const videoRef = useRef<HTMLVideoElement>(null);

    useImperativeHandle(ref, () => ({
      play: () => {
        videoRef.current?.play();
      },
      pause: () => {
        videoRef.current?.pause();
      },
      seek: (time: number) => {
        if (videoRef.current) {
          videoRef.current.currentTime = time;
        }
      },
      getCurrentTime: () => {
        return videoRef.current?.currentTime || 0;
      }
    }));

    return <video ref={videoRef} src={src} />;
  }
);

// Parent
function VideoApp() {
  const playerRef = useRef<VideoPlayerHandle>(null);

  return (
    <div>
      <VideoPlayer ref={playerRef} src="video.mp4" />
      <button onClick={() => playerRef.current?.play()}>Play</button>
      <button onClick={() => playerRef.current?.pause()}>Pause</button>
      <button onClick={() => playerRef.current?.seek(30)}>Skip to 30s</button>
    </div>
  );
}
```

**When to use:**
- Building component libraries with imperative APIs
- Focus management (modals, forms)
- Third-party library integration (canvas, video, maps)
- Animation triggers

**When NOT to use:**
- Prefer props and callbacks (declarative) over imperative methods
- Don't expose internal implementation details
- Use sparingly—React is declarative by design

---

### 3. **useLayoutEffect: Synchronous DOM Mutations**

**useEffect vs. useLayoutEffect:**

| Hook | Timing | Blocks Paint | Use Case |
|------|--------|--------------|----------|
| **useEffect** | After paint | No ❌ | Data fetching, subscriptions, most side effects |
| **useLayoutEffect** | Before paint | Yes ✅ | DOM measurements, preventing flicker |

**Timeline:**

```
1. React renders component
2. React commits to DOM
3. 🔵 useLayoutEffect runs (synchronous, blocks browser paint)
4. Browser paints screen
5. 🟢 useEffect runs (asynchronous, after paint)
```

**Example: Measuring DOM**

```jsx
function Tooltip({ children }: { children: React.ReactNode }) {
  const [position, setPosition] = useState({ top: 0, left: 0 });
  const tooltipRef = useRef<HTMLDivElement>(null);

  useLayoutEffect(() => {
    // Measure tooltip BEFORE browser paints
    if (tooltipRef.current) {
      const rect = tooltipRef.current.getBoundingClientRect();
      
      // Calculate position to keep tooltip in viewport
      const newTop = rect.top < 0 ? 0 : rect.top;
      const newLeft = rect.left + rect.width > window.innerWidth
        ? window.innerWidth - rect.width
        : rect.left;
      
      setPosition({ top: newTop, left: newLeft });
    }
  });

  return (
    <div 
      ref={tooltipRef}
      style={{ position: 'absolute', top: position.top, left: position.left }}
    >
      {children}
    </div>
  );
}
```

**Preventing flicker:**

```jsx
// ❌ BAD: useEffect causes flicker (element renders, then moves)
function Box() {
  const [height, setHeight] = useState(0);
  const ref = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (ref.current) {
      setHeight(ref.current.scrollHeight);  // Causes flicker!
    }
  }, []);

  return <div ref={ref} style={{ height }}>{/* content */}</div>;
}

// ✅ GOOD: useLayoutEffect runs before paint (no flicker)
function Box() {
  const [height, setHeight] = useState(0);
  const ref = useRef<HTMLDivElement>(null);

  useLayoutEffect(() => {
    if (ref.current) {
      setHeight(ref.current.scrollHeight);  // Runs before paint
    }
  }, []);

  return <div ref={ref} style={{ height }}>{/* content */}</div>;
}
```

**When to use useLayoutEffect:**
- Measuring DOM elements (getBoundingClientRect, scrollHeight, etc.)
- Preventing visual flicker
- Synchronizing with third-party libraries that mutate DOM
- Tooltip/popover positioning
- Reading layout before paint

**When NOT to use:**
- Data fetching (use useEffect)
- Subscriptions (use useEffect)
- Any side effect that doesn't need synchronous timing
- **Default to useEffect**, only use useLayoutEffect when you see visual issues

**Warning:** useLayoutEffect blocks painting, so it can hurt performance. Use sparingly.

---

### 4. **useDebugValue: Custom Hook Debugging**

**Purpose:** Display custom hook values in React DevTools.

```jsx
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(navigator.onLine);

  useEffect(() => {
    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);

    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);

    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  // Shows "OnlineStatus: online" or "OnlineStatus: offline" in DevTools
  useDebugValue(isOnline ? 'online' : 'offline');

  return isOnline;
}
```

**With formatting function (deferred expensive computation):**

```jsx
function useUser(userId: string) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);

  // Second argument: format function (only runs when DevTools is open)
  useDebugValue(user, (user) => {
    // Expensive formatting only happens in DevTools
    return user ? `User: ${user.name} (${user.email})` : 'No user';
  });

  return user;
}
```

**When to use:**
- Custom hooks that are part of a shared library
- Debugging complex hook composition
- Making hook internals visible in DevTools

**When NOT to use:**
- Simple hooks (adds noise)
- Production-only code (DevTools not needed)

---

## Common Pitfalls

### ❌ **Pitfall 1: Overusing useReducer**

```jsx
// BAD: useReducer for simple toggle
const [open, dispatch] = useReducer((state) => !state, false);

// GOOD: useState is simpler
const [open, setOpen] = useState(false);
```

**Rule of thumb:** If your reducer just sets a value, use useState.

---

### ❌ **Pitfall 2: Mutating state in reducer**

```jsx
// BAD: Mutates state directly
function reducer(state, action) {
  switch (action.type) {
    case 'ADD_ITEM':
      state.items.push(action.payload);  // ❌ MUTATION!
      return state;
  }
}

// GOOD: Returns new state
function reducer(state, action) {
  switch (action.type) {
    case 'ADD_ITEM':
      return {
        ...state,
        items: [...state.items, action.payload]  // ✅ New array
      };
  }
}
```

---

### ❌ **Pitfall 3: Using useLayoutEffect unnecessarily**

```jsx
// BAD: useLayoutEffect for data fetching (blocks paint!)
useLayoutEffect(() => {
  fetchData().then(setData);
}, []);

// GOOD: useEffect for async operations
useEffect(() => {
  fetchData().then(setData);
}, []);
```

---

### ❌ **Pitfall 4: Exposing too much with useImperativeHandle**

```jsx
// BAD: Exposes internal implementation
useImperativeHandle(ref, () => ({
  inputRef,  // ❌ Exposes internal ref
  state,     // ❌ Exposes internal state
  handleChange,  // ❌ Exposes internal handler
}));

// GOOD: Minimal, focused API
useImperativeHandle(ref, () => ({
  focus: () => inputRef.current?.focus(),  // ✅ Focused method
  clear: () => setValue('')  // ✅ Controlled method
}));
```

---

## Quick Self-Check

✅ **You understand advanced hooks if you can:**

1. Explain when useReducer is better than useState
2. Write a reducer for a shopping cart
3. Use useImperativeHandle to expose component methods
4. Explain the difference between useEffect and useLayoutEffect
5. Identify when useLayoutEffect is needed (and when it's not)
6. Use useDebugValue to debug custom hooks
7. Combine multiple advanced hooks in a custom hook
8. Avoid common pitfalls (mutations, unnecessary useLayoutEffect)

---

## Interview Questions to Practice

**Beginner:**
1. What's the difference between useState and useReducer?
2. When would you use useReducer instead of useState?
3. What does useImperativeHandle do?

**Intermediate:**
4. Explain the difference between useEffect and useLayoutEffect
5. How would you implement a shopping cart with useReducer?
6. When would you use useLayoutEffect vs. useEffect?
7. What's the signature of a reducer function?

**Advanced:**
8. Build a multi-step form wizard with useReducer
9. Create a custom video player component with useImperativeHandle
10. Explain how useLayoutEffect prevents visual flicker

---

## Further Practice

**Build these:**
1. **Shopping cart** with useReducer (add, remove, update quantity)
2. **Multi-step form wizard** with useReducer (validate each step)
3. **Custom input component** with useImperativeHandle (focus, clear, validate methods)
4. **Tooltip component** with useLayoutEffect (positioning)
5. **Undo/redo** with useReducer (command pattern)
6. **Finite state machine** with useReducer (traffic light, game states)

**Resources:**
- [useReducer documentation](https://react.dev/reference/react/useReducer)
- [useImperativeHandle documentation](https://react.dev/reference/react/useImperativeHandle)
- [useLayoutEffect documentation](https://react.dev/reference/react/useLayoutEffect)

---

## Summary

**useReducer:**
- Use for complex state logic with multiple actions
- Centralizes state update logic
- Better for state with multiple related values
- Signature: `const [state, dispatch] = useReducer(reducer, initialState)`

**useImperativeHandle:**
- Exposes component methods to parent via ref
- Use with forwardRef
- Use sparingly (prefer declarative props)
- Good for: focus management, imperative animations, library integration

**useLayoutEffect:**
- Runs synchronously before browser paint
- Use for: DOM measurements, preventing flicker
- Blocks painting (performance cost)
- **Default to useEffect**, only use when you see visual problems

**useDebugValue:**
- Displays custom hook values in React DevTools
- Second argument defers expensive formatting
- Use in shared/library hooks

**Remember:** Advanced hooks solve specific problems. Don't reach for them until you've identified the problem they solve. useState + useEffect covers 80% of cases.
