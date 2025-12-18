# Modern React Patterns (2024-2025)

**Level:** ⚡ React 19+ Features  
**Tags:** `react-19`, `use-hook`, `actions`, `useActionState`, `useOptimistic`, `modern-patterns`

---

## Why this matters

React 19 introduces revolutionary new patterns that change how we handle async data, forms, and state updates. The `use()` hook, Actions, `useActionState`, and `useOptimistic` represent the future of React development. Understanding these patterns is critical for staying current and demonstrates mastery of cutting-edge React features.

**Why interviewers care:**
- Modern patterns show you're keeping up with React's evolution
- Tests deep understanding of async rendering and Suspense
- Reveals ability to write cleaner, more maintainable code
- Demonstrates knowledge of progressive enhancement
- Shows understanding of React's architectural direction

**Real-world implications:**
- **Simpler async code:** `use()` eliminates complex useEffect patterns
- **Better forms:** Actions provide built-in loading/error states
- **Instant feedback:** `useOptimistic` creates responsive UIs
- **Less boilerplate:** Modern patterns reduce code complexity
- **Progressive enhancement:** Forms work without JavaScript

**What changed in React 19:**
- New `use()` hook for reading promises and context
- Actions pattern for async transitions
- `useActionState` for form state management
- `useOptimistic` for optimistic UI updates
- Context as provider (`<Context>` instead of `<Context.Provider>`)
- Ref as prop (no more `forwardRef`)
- Improved hydration error messages

**What you must know:**
- `use()` hook for promises and context
- Actions vs. regular async functions
- `useActionState` for forms
- `useOptimistic` for instant UI updates
- When to use each pattern
- How they work together

**Interview red flags:** Not knowing React 19's new features, or dismissing them as "just syntactic sugar" without understanding the architectural improvements.

---

## Core Ideas

### 1. **The use() Hook: Reading Promises in Render**

**Problem:** Before React 19, reading promises required complex useEffect patterns.

**Before React 19:**

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    setLoading(true);
    fetchUser(userId)
      .then(data => {
        setUser(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err);
        setLoading(false);
      });
  }, [userId]);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  return <div>{user.name}</div>;
}
```

**With React 19 use():**

```jsx
import { use, Suspense } from 'react';

function UserProfile({ userPromise }) {
  // use() unwraps the promise
  const user = use(userPromise);
  
  return <div>{user.name}</div>;
}

function Page() {
  // Create promise
  const userPromise = fetchUser('123');
  
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <UserProfile userPromise={userPromise} />
    </Suspense>
  );
}
```

**Key benefits:**
- No useState/useEffect boilerplate
- Works with Suspense automatically
- Can be called conditionally (unlike hooks!)
- Cleaner error boundaries

**Reading context with use():**

```jsx
import { use, createContext } from 'react';

const ThemeContext = createContext('light');

function ThemedButton() {
  // use() can read context (alternative to useContext)
  const theme = use(ThemeContext);
  
  return <button className={theme}>Click me</button>;
}

// Can be called conditionally!
function ConditionalTheme({ useTheme }) {
  let theme = 'default';
  
  if (useTheme) {
    theme = use(ThemeContext);  // ✅ Conditional - not possible with useContext
  }
  
  return <div className={theme}>Content</div>;
}
```

**use() vs. useContext/useEffect:**

```jsx
// ❌ Old way: Can't call conditionally
function Component({ needsTheme }) {
  // Can't do this with useContext
  if (needsTheme) {
    const theme = useContext(ThemeContext);  // Error: Conditional hook call
  }
}

// ✅ New way: Conditional use()
function Component({ needsTheme }) {
  if (needsTheme) {
    const theme = use(ThemeContext);  // ✅ Works!
  }
}
```

---

### 2. **Actions: Async State Transitions**

**Actions** are async functions that wrap state updates in transitions automatically.

**Regular async function (not an Action):**

```jsx
function Component() {
  const [data, setData] = useState(null);
  const [isPending, startTransition] = useTransition();
  
  const handleClick = async () => {
    startTransition(async () => {  // Manual transition
      const result = await fetchData();
      setData(result);
    });
  };
  
  return (
    <button onClick={handleClick} disabled={isPending}>
      {isPending ? 'Loading...' : 'Fetch Data'}
    </button>
  );
}
```

**With Actions (automatic transition):**

```jsx
function Component() {
  const [data, setData] = useState(null);
  
  // Action: async function passed to event handler or form action
  const handleClick = async () => {
    // React automatically wraps this in a transition!
    const result = await fetchData();
    setData(result);
  };
  
  return (
    <button onClick={handleClick}>
      Fetch Data
    </button>
  );
}
```

**What makes a function an Action:**
1. Async function
2. Used in event handler (`onClick`, `onSubmit`, etc.) OR
3. Used as form `action` prop

**Actions provide:**
- Automatic pending state (`useActionState` or `useOptimistic`)
- Error handling
- Automatic transitions (keep UI responsive)
- Integration with Suspense

**Example: Form Action**

```jsx
'use client';

function TodoForm({ addTodo }) {
  // addTodo is an Action (async function)
  return (
    <form action={addTodo}>
      <input name="title" required />
      <button type="submit">Add</button>
    </form>
  );
}
```

---

### 3. **useActionState: Form State Management**

**useActionState** manages form submissions with built-in loading and error states.

**Signature:**

```jsx
const [state, formAction, isPending] = useActionState(action, initialState);
```

- `state` - Current state (return value from action)
- `formAction` - Function to pass to form's `action` prop
- `isPending` - Boolean indicating if action is running

**Basic example:**

```jsx
'use client';

import { useActionState } from 'react';
import { createTodo } from '@/app/actions';

function TodoForm() {
  const [state, formAction, isPending] = useActionState(
    createTodo,
    { error: null }  // Initial state
  );
  
  return (
    <form action={formAction}>
      <input name="title" required />
      
      {/* Show errors */}
      {state?.error && (
        <p className="error">{state.error}</p>
      )}
      
      {/* Show loading state */}
      <button type="submit" disabled={isPending}>
        {isPending ? 'Creating...' : 'Create Todo'}
      </button>
    </form>
  );
}
```

**Server Action (app/actions.ts):**

```jsx
'use server';

export async function createTodo(prevState, formData) {
  const title = formData.get('title');
  
  // Validation
  if (!title || title.length < 3) {
    return { error: 'Title must be at least 3 characters' };
  }
  
  try {
    await db.todo.create({ data: { title } });
    return { success: true };
  } catch (error) {
    return { error: 'Failed to create todo' };
  }
}
```

**With validation feedback:**

```jsx
'use client';

import { useActionState } from 'react';
import { createUser } from '@/app/actions';

function SignupForm() {
  const [state, formAction, isPending] = useActionState(
    createUser,
    { errors: {} }
  );
  
  return (
    <form action={formAction}>
      <div>
        <label htmlFor="email">Email</label>
        <input id="email" name="email" type="email" required />
        {state?.errors?.email && (
          <span className="error">{state.errors.email}</span>
        )}
      </div>
      
      <div>
        <label htmlFor="password">Password</label>
        <input id="password" name="password" type="password" required />
        {state?.errors?.password && (
          <span className="error">{state.errors.password}</span>
        )}
      </div>
      
      {state?.errors?.general && (
        <p className="error">{state.errors.general}</p>
      )}
      
      <button type="submit" disabled={isPending}>
        {isPending ? 'Creating account...' : 'Sign Up'}
      </button>
      
      {state?.success && (
        <p className="success">Account created!</p>
      )}
    </form>
  );
}
```

**Server Action with detailed validation:**

```jsx
'use server';

import { z } from 'zod';

const userSchema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(8, 'Password must be 8+ characters')
});

export async function createUser(prevState, formData) {
  const result = userSchema.safeParse({
    email: formData.get('email'),
    password: formData.get('password')
  });
  
  if (!result.success) {
    return {
      errors: result.error.flatten().fieldErrors
    };
  }
  
  try {
    await db.user.create({ data: result.data });
    return { success: true };
  } catch (error) {
    return { errors: { general: 'Failed to create account' } };
  }
}
```

---

### 4. **useOptimistic: Instant UI Updates**

**useOptimistic** shows immediate UI changes while async operations are pending.

**Problem:** Users wait for server response before seeing changes.

**Solution:** Show optimistic update immediately, revert if fails.

**Basic example:**

```jsx
'use client';

import { useOptimistic } from 'react';
import { sendMessage } from '@/app/actions';

function ChatThread({ messages }) {
  const [optimisticMessages, addOptimisticMessage] = useOptimistic(
    messages,
    (state, newMessage) => [...state, newMessage]
  );
  
  const handleSend = async (formData) => {
    const text = formData.get('message');
    
    // Add message optimistically (instant UI update)
    addOptimisticMessage({
      id: crypto.randomUUID(),
      text,
      sending: true  // Mark as pending
    });
    
    // Send to server (runs in background)
    await sendMessage(formData);
  };
  
  return (
    <div>
      {optimisticMessages.map(msg => (
        <div 
          key={msg.id}
          className={msg.sending ? 'opacity-50' : ''}
        >
          {msg.text}
          {msg.sending && ' (sending...)'}
        </div>
      ))}
      
      <form action={handleSend}>
        <input name="message" required />
        <button type="submit">Send</button>
      </form>
    </div>
  );
}
```

**Signature:**

```jsx
const [optimisticState, addOptimistic] = useOptimistic(
  actualState,
  updateFn
);
```

- `actualState` - Current actual data (from props/state)
- `updateFn` - `(state, optimisticValue) => newState`
- Returns: `[optimisticState, addOptimistic]`

**With multiple operations:**

```jsx
'use client';

import { useOptimistic } from 'react';
import { addTodo, deleteTodo, toggleTodo } from '@/app/actions';

function TodoList({ initialTodos }) {
  const [optimisticTodos, updateOptimisticTodos] = useOptimistic(
    initialTodos,
    (state, { action, id, todo }) => {
      switch (action) {
        case 'add':
          return [...state, todo];
        case 'delete':
          return state.filter(t => t.id !== id);
        case 'toggle':
          return state.map(t => 
            t.id === id ? { ...t, completed: !t.completed } : t
          );
        default:
          return state;
      }
    }
  );
  
  const handleAdd = async (formData) => {
    const title = formData.get('title');
    const tempId = crypto.randomUUID();
    
    updateOptimisticTodos({
      action: 'add',
      todo: { id: tempId, title, completed: false, pending: true }
    });
    
    await addTodo(formData);
  };
  
  const handleDelete = async (id) => {
    updateOptimisticTodos({ action: 'delete', id });
    await deleteTodo(id);
  };
  
  const handleToggle = async (id) => {
    updateOptimisticTodos({ action: 'toggle', id });
    await toggleTodo(id);
  };
  
  return (
    <div>
      <form action={handleAdd}>
        <input name="title" required />
        <button type="submit">Add</button>
      </form>
      
      <ul>
        {optimisticTodos.map(todo => (
          <li 
            key={todo.id}
            className={todo.pending ? 'opacity-50' : ''}
          >
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => handleToggle(todo.id)}
            />
            {todo.title}
            <button onClick={() => handleDelete(todo.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

**Error handling with rollback:**

```jsx
'use client';

import { useOptimistic, useState } from 'react';
import { likeTweet } from '@/app/actions';

function Tweet({ tweet }) {
  const [error, setError] = useState(null);
  const [optimisticLikes, addOptimisticLike] = useOptimistic(
    tweet.likes,
    (currentLikes, amount) => currentLikes + amount
  );
  
  const handleLike = async () => {
    setError(null);
    
    // Optimistic update
    addOptimisticLike(1);
    
    try {
      await likeTweet(tweet.id);
    } catch (err) {
      // On error, optimistic update automatically reverts
      setError('Failed to like tweet');
    }
  };
  
  return (
    <div>
      <p>{tweet.text}</p>
      <button onClick={handleLike}>
        ❤️ {optimisticLikes}
      </button>
      {error && <p className="error">{error}</p>}
    </div>
  );
}
```

---

### 5. **Combining Patterns**

**use() + Actions + useOptimistic:**

```jsx
'use client';

import { use, useOptimistic, Suspense } from 'react';

// Component that reads promise
function TodoList({ todosPromise }) {
  const initialTodos = use(todosPromise);  // Read promise
  
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    initialTodos,
    (state, newTodo) => [...state, newTodo]
  );
  
  // Action
  const addTodo = async (formData) => {
    const title = formData.get('title');
    
    // Optimistic update
    addOptimisticTodo({
      id: crypto.randomUUID(),
      title,
      pending: true
    });
    
    // Server mutation
    await createTodo(formData);
  };
  
  return (
    <div>
      <form action={addTodo}>
        <input name="title" required />
        <button type="submit">Add</button>
      </form>
      
      <ul>
        {optimisticTodos.map(todo => (
          <li key={todo.id} className={todo.pending ? 'opacity-50' : ''}>
            {todo.title}
          </li>
        ))}
      </ul>
    </div>
  );
}

// Parent with Suspense
function Page() {
  const todosPromise = fetchTodos();
  
  return (
    <Suspense fallback={<div>Loading todos...</div>}>
      <TodoList todosPromise={todosPromise} />
    </Suspense>
  );
}
```

**useActionState + useOptimistic:**

```jsx
'use client';

import { useActionState, useOptimistic } from 'react';
import { createComment } from '@/app/actions';

function CommentSection({ initialComments }) {
  const [state, formAction, isPending] = useActionState(
    createComment,
    { error: null }
  );
  
  const [optimisticComments, addOptimisticComment] = useOptimistic(
    initialComments,
    (state, newComment) => [...state, newComment]
  );
  
  const handleSubmit = async (formData) => {
    const text = formData.get('text');
    
    // Optimistic update
    addOptimisticComment({
      id: crypto.randomUUID(),
      text,
      author: 'You',
      pending: true
    });
    
    // Clear form
    formData.set('text', '');
  };
  
  return (
    <div>
      <ul>
        {optimisticComments.map(comment => (
          <li key={comment.id} className={comment.pending ? 'opacity-50' : ''}>
            <strong>{comment.author}:</strong> {comment.text}
          </li>
        ))}
      </ul>
      
      <form action={formAction} onSubmit={handleSubmit}>
        <textarea name="text" required />
        {state?.error && <p className="error">{state.error}</p>}
        <button type="submit" disabled={isPending}>
          {isPending ? 'Posting...' : 'Post Comment'}
        </button>
      </form>
    </div>
  );
}
```

---

### 6. **Other React 19 Improvements**

**Ref as prop (no more forwardRef):**

```jsx
// ❌ Old way: forwardRef
const Input = forwardRef((props, ref) => {
  return <input ref={ref} {...props} />;
});

// ✅ New way: ref is just a prop
function Input({ ref, ...props }) {
  return <input ref={ref} {...props} />;
}

// Usage (same)
function Form() {
  const inputRef = useRef();
  return <Input ref={inputRef} />;
}
```

**Context as provider:**

```jsx
// ❌ Old way: Context.Provider
const ThemeContext = createContext('light');

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Page />
    </ThemeContext.Provider>
  );
}

// ✅ New way: Context directly
function App() {
  return (
    <ThemeContext value="dark">
      <Page />
    </ThemeContext>
  );
}
```

**Document metadata (built-in):**

```jsx
// ❌ Old way: react-helmet or Next.js Head
import Head from 'next/head';

function Page() {
  return (
    <>
      <Head>
        <title>My Page</title>
        <meta name="description" content="..." />
      </Head>
      <div>Content</div>
    </>
  );
}

// ✅ New way: Built-in components
function Page() {
  return (
    <>
      <title>My Page</title>
      <meta name="description" content="..." />
      <div>Content</div>
    </>
  );
}
```

**Improved hydration errors:**

```jsx
// React 19 provides much better error messages when hydration fails
// Before: "Text content did not match" (not helpful)
// Now: Shows exact mismatch with source location and suggestions
```

---

## Common Pitfalls

### ❌ **Pitfall 1: Using use() outside render**

```jsx
// ❌ Can't use use() outside component
const data = use(promise);  // Error!

function Component() {
  return <div>{data}</div>;
}

// ✅ Call inside component
function Component({ dataPromise }) {
  const data = use(dataPromise);  // ✅ Correct
  return <div>{data}</div>;
}
```

---

### ❌ **Pitfall 2: Forgetting Suspense with use()**

```jsx
// ❌ No Suspense boundary - throws error
function Page() {
  return <DataComponent dataPromise={fetchData()} />;
}

// ✅ Wrap in Suspense
function Page() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <DataComponent dataPromise={fetchData()} />
    </Suspense>
  );
}
```

---

### ❌ **Pitfall 3: Not handling optimistic update failures**

```jsx
// ❌ No error handling - user doesn't know update failed
const handleLike = async () => {
  addOptimisticLike(1);
  await likeTweet(id);  // Fails silently
};

// ✅ Handle errors
const handleLike = async () => {
  addOptimisticLike(1);
  try {
    await likeTweet(id);
  } catch (error) {
    setError('Failed to like');
    // Optimistic update auto-reverts
  }
};
```

---

## Quick Self-Check

✅ **You understand modern React patterns if you can:**

1. Use `use()` to read promises in components
2. Explain what Actions are
3. Use `useActionState` for form state
4. Implement optimistic updates with `useOptimistic`
5. Combine `use()` + Actions + `useOptimistic`
6. Use ref as prop (no forwardRef)
7. Use Context as provider
8. Understand when to use each pattern
9. Handle errors in optimistic updates
10. Know the benefits of each new pattern

---

## Interview Questions to Practice

**Beginner:**
1. What is the `use()` hook?
2. What's the difference between `use()` and `useEffect`?
3. What are Actions in React 19?

**Intermediate:**
4. How does `useActionState` differ from `useState`?
5. When would you use `useOptimistic`?
6. How do you handle errors with optimistic updates?
7. What's the benefit of ref as prop vs. forwardRef?

**Advanced:**
8. How would you implement a real-time chat with optimistic updates?
9. Explain how `use()`, Actions, and `useOptimistic` work together
10. When should you use `use()` vs. Server Components?

---

## Further Practice

**Build these:**
1. **Chat app** with optimistic messages
2. **Todo app** with `use()` + `useOptimistic`
3. **Like button** with optimistic updates
4. **Multi-step form** with `useActionState`
5. **Comment system** combining all patterns

**Resources:**
- [use() hook docs](https://react.dev/reference/react/use)
- [useActionState docs](https://react.dev/reference/react/useActionState)
- [useOptimistic docs](https://react.dev/reference/react/useOptimistic)
- [React 19 announcement](https://react.dev/blog/2024/04/25/react-19)

---

## Summary

**use() hook:**
- Reads promises and context
- Can be called conditionally (unlike hooks)
- Works with Suspense
- Simplifies async data fetching

**Actions:**
- Async functions with automatic transitions
- Used in event handlers and forms
- Provide built-in pending states
- Work with `useActionState` and `useOptimistic`

**useActionState:**
- Manages form state and submissions
- Provides loading and error states
- Signature: `[state, formAction, isPending]`
- Perfect for forms with validation

**useOptimistic:**
- Shows instant UI updates
- Reverts on failure automatically
- Signature: `[optimisticState, addOptimistic]`
- Great for likes, comments, todos

**Other improvements:**
- Ref as prop (no forwardRef)
- Context as provider (no .Provider)
- Built-in metadata components
- Better hydration errors

**Remember:** These patterns work together. Use `use()` for data fetching, Actions for mutations, `useActionState` for forms, and `useOptimistic` for instant feedback.
