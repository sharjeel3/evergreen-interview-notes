# State Management Patterns

**Level:** 🔥 Advanced  
**Tags:** `state-management`, `redux`, `zustand`, `jotai`, `recoil`, `context`, `architecture`

---

## Why this matters

State management is one of the most critical architectural decisions in React applications. Interviewers use state management questions to assess your understanding of application architecture, scalability, and ability to choose the right tool for the job. This topic separates junior developers who only know useState from senior engineers who can design complex state architectures.

**Why interviewers care:**
- State management reveals architectural thinking and ability to scale applications
- Choice of state library shows understanding of trade-offs (complexity vs. power)
- Tests knowledge of performance implications (re-renders, subscriptions)
- Demonstrates experience with real-world production applications
- Senior roles expect knowledge of multiple state management approaches
- Migration strategies show practical experience and decision-making skills

**Real-world implications:**
- **App scalability:** Poor state management makes large apps unmaintainable
- **Performance:** Inefficient state updates cause unnecessary re-renders
- **Developer experience:** Good state management makes code easier to understand
- **Team collaboration:** Shared state patterns improve team productivity
- **Debugging:** Proper state management makes bugs easier to track down
- **Testing:** Well-structured state is easier to test

**Common state management challenges:**
- Prop drilling (passing props through many components)
- Global state (user data, theme, auth)
- Shared state (multiple components need same data)
- Server state (data from APIs)
- Computed/derived state
- Optimistic updates
- State synchronization across tabs/windows

**What you must know:**
- When to use local state (useState) vs. global state
- Context API (built-in, good for simple global state)
- Redux (industry standard, powerful but verbose)
- Zustand (simple, modern alternative to Redux)
- Jotai/Recoil (atomic state management)
- When to use each solution
- Server state libraries (React Query, SWR)

**Interview red flags:** Using Redux for everything, not knowing alternatives, or dismissing Context API without understanding when it's appropriate.

---

## Core Ideas

### 1. **The State Management Spectrum**

**Local State (useState) - Start here:**

```jsx
// Good for: Component-specific state
function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

**When to use:** State only needed in one component (or passed to 1-2 children).

**Lifted State - Next step:**

```jsx
// Good for: Sharing between siblings
function Parent() {
  const [count, setCount] = useState(0);
  
  return (
    <>
      <Counter count={count} setCount={setCount} />
      <Display count={count} />
    </>
  );
}
```

**When to use:** State needed by multiple components with a common parent.

**Context API - For simple global state:**

```jsx
// Good for: Theme, auth, simple global state
const ThemeContext = createContext();

function App() {
  const [theme, setTheme] = useState('light');
  
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <Page />
    </ThemeContext.Provider>
  );
}

function ThemedButton() {
  const { theme } = useContext(ThemeContext);
  return <button className={theme}>Click me</button>;
}
```

**When to use:** Avoiding prop drilling, infrequently changing state, small apps.

**State Management Library - For complex apps:**

```jsx
// Good for: Complex state, many components, frequent updates
// Redux, Zustand, Jotai, etc.
```

**When to use:** Large apps, complex state logic, high-frequency updates.

---

### 2. **Context API: Built-in Global State**

**Basic Context:**

```jsx
// context/UserContext.tsx
import { createContext, useContext, useState } from 'react';

interface User {
  id: string;
  name: string;
  email: string;
}

interface UserContextType {
  user: User | null;
  login: (user: User) => void;
  logout: () => void;
}

const UserContext = createContext<UserContextType | undefined>(undefined);

export function UserProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);

  const login = (user: User) => setUser(user);
  const logout = () => setUser(null);

  return (
    <UserContext.Provider value={{ user, login, logout }}>
      {children}
    </UserContext.Provider>
  );
}

export function useUser() {
  const context = useContext(UserContext);
  if (!context) {
    throw new Error('useUser must be used within UserProvider');
  }
  return context;
}
```

```jsx
// App.tsx
function App() {
  return (
    <UserProvider>
      <Header />
      <Main />
    </UserProvider>
  );
}

// Any component can use it
function Header() {
  const { user, logout } = useUser();
  
  return (
    <header>
      {user ? (
        <>
          <span>Welcome, {user.name}</span>
          <button onClick={logout}>Logout</button>
        </>
      ) : (
        <span>Please login</span>
      )}
    </header>
  );
}
```

**Performance optimization with useMemo:**

```jsx
export function UserProvider({ children }) {
  const [user, setUser] = useState(null);

  // Memoize value to prevent re-renders
  const value = useMemo(() => ({
    user,
    login: (user) => setUser(user),
    logout: () => setUser(null)
  }), [user]);

  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
}
```

**Multiple contexts (composition):**

```jsx
function App() {
  return (
    <UserProvider>
      <ThemeProvider>
        <SettingsProvider>
          <Page />
        </SettingsProvider>
      </ThemeProvider>
    </UserProvider>
  );
}

// Or use a wrapper
function AppProviders({ children }) {
  return (
    <UserProvider>
      <ThemeProvider>
        <SettingsProvider>
          {children}
        </SettingsProvider>
      </ThemeProvider>
    </UserProvider>
  );
}
```

**Pros:**
- Built into React (no dependencies)
- Simple API
- Good for infrequently changing state
- Perfect for theme, auth, locale

**Cons:**
- Performance issues with frequent updates (all consumers re-render)
- Can lead to "provider hell" (many nested providers)
- No built-in DevTools
- No middleware/plugins

---

### 3. **Redux: The Industry Standard**

**Redux with Redux Toolkit (modern approach):**

```jsx
// store/userSlice.ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface User {
  id: string;
  name: string;
}

interface UserState {
  currentUser: User | null;
  isLoading: boolean;
  error: string | null;
}

const initialState: UserState = {
  currentUser: null,
  isLoading: false,
  error: null
};

const userSlice = createSlice({
  name: 'user',
  initialState,
  reducers: {
    setUser: (state, action: PayloadAction<User>) => {
      state.currentUser = action.payload;
    },
    clearUser: (state) => {
      state.currentUser = null;
    },
    setLoading: (state, action: PayloadAction<boolean>) => {
      state.isLoading = action.payload;
    },
    setError: (state, action: PayloadAction<string>) => {
      state.error = action.payload;
    }
  }
});

export const { setUser, clearUser, setLoading, setError } = userSlice.actions;
export default userSlice.reducer;
```

```jsx
// store/index.ts
import { configureStore } from '@reduxjs/toolkit';
import userReducer from './userSlice';
import cartReducer from './cartSlice';

export const store = configureStore({
  reducer: {
    user: userReducer,
    cart: cartReducer
  }
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

```jsx
// App.tsx
import { Provider } from 'react-redux';
import { store } from './store';

function App() {
  return (
    <Provider store={store}>
      <Page />
    </Provider>
  );
}
```

```jsx
// components/UserProfile.tsx
import { useSelector, useDispatch } from 'react-redux';
import { RootState } from '@/store';
import { setUser, clearUser } from '@/store/userSlice';

function UserProfile() {
  const user = useSelector((state: RootState) => state.user.currentUser);
  const dispatch = useDispatch();

  const handleLogout = () => {
    dispatch(clearUser());
  };

  return user ? (
    <div>
      <p>{user.name}</p>
      <button onClick={handleLogout}>Logout</button>
    </div>
  ) : null;
}
```

**Async actions with createAsyncThunk:**

```jsx
// store/userSlice.ts
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

export const fetchUser = createAsyncThunk(
  'user/fetchUser',
  async (userId: string) => {
    const response = await fetch(`/api/users/${userId}`);
    return response.json();
  }
);

const userSlice = createSlice({
  name: 'user',
  initialState,
  reducers: {
    // ... sync actions
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUser.pending, (state) => {
        state.isLoading = true;
      })
      .addCase(fetchUser.fulfilled, (state, action) => {
        state.isLoading = false;
        state.currentUser = action.payload;
      })
      .addCase(fetchUser.rejected, (state, action) => {
        state.isLoading = false;
        state.error = action.error.message || 'Failed to fetch user';
      });
  }
});
```

```jsx
// Usage
function UserProfile() {
  const dispatch = useDispatch();
  
  useEffect(() => {
    dispatch(fetchUser('123'));
  }, [dispatch]);
}
```

**Pros:**
- Industry standard (most jobs know Redux)
- Excellent DevTools
- Time-travel debugging
- Middleware ecosystem
- Predictable state updates
- Great for large teams

**Cons:**
- Verbose (lots of boilerplate, even with Toolkit)
- Learning curve
- Overkill for small apps
- Can be over-engineered

**When to use Redux:**
- Large applications with complex state
- Many developers on team (shared patterns)
- Need advanced debugging (time-travel)
- Existing Redux codebase
- Need middleware (logging, analytics)

---

### 4. **Zustand: Simple Modern Alternative**

**Basic Zustand store:**

```jsx
// store/useUserStore.ts
import { create } from 'zustand';

interface User {
  id: string;
  name: string;
}

interface UserStore {
  user: User | null;
  isLoading: boolean;
  error: string | null;
  setUser: (user: User) => void;
  clearUser: () => void;
  fetchUser: (id: string) => Promise<void>;
}

export const useUserStore = create<UserStore>((set) => ({
  user: null,
  isLoading: false,
  error: null,
  
  setUser: (user) => set({ user }),
  
  clearUser: () => set({ user: null }),
  
  fetchUser: async (id) => {
    set({ isLoading: true, error: null });
    try {
      const response = await fetch(`/api/users/${id}`);
      const user = await response.json();
      set({ user, isLoading: false });
    } catch (error) {
      set({ error: 'Failed to fetch user', isLoading: false });
    }
  }
}));
```

```jsx
// components/UserProfile.tsx
import { useUserStore } from '@/store/useUserStore';

function UserProfile() {
  // Subscribe to specific parts of state
  const user = useUserStore((state) => state.user);
  const clearUser = useUserStore((state) => state.clearUser);
  
  return user ? (
    <div>
      <p>{user.name}</p>
      <button onClick={clearUser}>Logout</button>
    </div>
  ) : null;
}

// Or use entire store
function AnotherComponent() {
  const { user, isLoading, fetchUser } = useUserStore();
  
  useEffect(() => {
    fetchUser('123');
  }, [fetchUser]);
  
  if (isLoading) return <div>Loading...</div>;
  return <div>{user?.name}</div>;
}
```

**Zustand with middleware:**

```jsx
import { create } from 'zustand';
import { persist, devtools } from 'zustand/middleware';

export const useUserStore = create(
  devtools(
    persist(
      (set) => ({
        user: null,
        setUser: (user) => set({ user }),
        clearUser: () => set({ user: null })
      }),
      {
        name: 'user-storage'  // localStorage key
      }
    )
  )
);
```

**Pros:**
- Minimal boilerplate (easiest to learn)
- No provider needed
- Excellent TypeScript support
- Small bundle size (~1KB)
- Can use outside React components
- Fast (only re-renders subscribers)

**Cons:**
- Less ecosystem than Redux
- Fewer learning resources
- No official DevTools (though middleware exists)
- Newer library (less battle-tested)

**When to use Zustand:**
- Small to medium apps
- Want simplicity
- Don't need Redux ecosystem
- Migrating from Redux (less boilerplate)
- Rapid prototyping

---

### 5. **Jotai: Atomic State Management**

**Atoms (atomic pieces of state):**

```jsx
// store/atoms.ts
import { atom } from 'jotai';

interface User {
  id: string;
  name: string;
}

// Primitive atoms
export const userAtom = atom<User | null>(null);
export const themeAtom = atom<'light' | 'dark'>('light');
export const countAtom = atom(0);

// Derived atoms (computed)
export const userNameAtom = atom((get) => {
  const user = get(userAtom);
  return user?.name || 'Guest';
});

// Write-only atoms (actions)
export const incrementAtom = atom(
  null,  // no read
  (get, set) => {
    set(countAtom, get(countAtom) + 1);
  }
);

// Async atoms
export const userAsyncAtom = atom(async (get) => {
  const response = await fetch('/api/user');
  return response.json();
});
```

```jsx
// App.tsx
import { Provider } from 'jotai';

function App() {
  return (
    <Provider>
      <Page />
    </Provider>
  );
}
```

```jsx
// components/UserProfile.tsx
import { useAtom, useAtomValue, useSetAtom } from 'jotai';
import { userAtom, userNameAtom } from '@/store/atoms';

function UserProfile() {
  // Read and write
  const [user, setUser] = useAtom(userAtom);
  
  // Read only
  const userName = useAtomValue(userNameAtom);
  
  // Write only
  const setUser = useSetAtom(userAtom);
  
  return (
    <div>
      <p>Hello, {userName}</p>
      <button onClick={() => setUser(null)}>Logout</button>
    </div>
  );
}
```

**Pros:**
- Minimal boilerplate
- Excellent TypeScript support
- Bottom-up approach (compose atoms)
- Suspense support
- Derived state is easy
- Only re-renders what changed

**Cons:**
- Different mental model (atoms vs. store)
- Smaller community than Redux/Zustand
- Can be confusing for beginners
- Debugging can be harder

**When to use Jotai:**
- Want fine-grained reactivity
- Lots of derived/computed state
- Want Suspense integration
- Bottom-up state composition
- Small focused pieces of state

---

### 6. **Server State: React Query / TanStack Query**

**Note:** Server state (API data) is different from client state (UI state).

```jsx
// Traditional approach (client state for server data)
function UserList() {
  const [users, setUsers] = useState([]);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetch('/api/users')
      .then(res => res.json())
      .then(data => {
        setUsers(data);
        setIsLoading(false);
      })
      .catch(err => {
        setError(err);
        setIsLoading(false);
      });
  }, []);
  
  // Need to manually handle refetch, cache, etc.
}
```

```jsx
// React Query approach (server state library)
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

function UserList() {
  // Automatic loading, error, caching, refetching
  const { data: users, isLoading, error } = useQuery({
    queryKey: ['users'],
    queryFn: () => fetch('/api/users').then(res => res.json())
  });
  
  const queryClient = useQueryClient();
  
  const createUser = useMutation({
    mutationFn: (newUser) => 
      fetch('/api/users', {
        method: 'POST',
        body: JSON.stringify(newUser)
      }),
    onSuccess: () => {
      // Invalidate and refetch
      queryClient.invalidateQueries({ queryKey: ['users'] });
    }
  });
  
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  
  return (
    <div>
      {users.map(user => <div key={user.id}>{user.name}</div>)}
      <button onClick={() => createUser.mutate({ name: 'New User' })}>
        Add User
      </button>
    </div>
  );
}
```

**When to use React Query/SWR:**
- Fetching data from APIs
- Need caching, refetching, pagination
- Optimistic updates
- Background sync
- Don't want to manage server state manually

---

### 7. **Decision Tree: Which State Solution?**

```
START: Do you need state?
  ├─ No → Use props
  └─ Yes → Continue

Is it UI state or server state?
  ├─ Server state (API data) → Use React Query/SWR
  └─ UI state → Continue

How many components need it?
  ├─ One component → useState
  ├─ Parent + children → Lifted state
  └─ Many unrelated components → Continue

How complex is the state logic?
  ├─ Simple (theme, auth) → Context API
  └─ Complex → Continue

How big is your app?
  ├─ Small/Medium → Zustand or Jotai
  └─ Large/Enterprise → Redux (or Zustand)

Do you need Redux specifically?
  ├─ Yes (existing codebase, team standard) → Redux Toolkit
  ├─ Want simplicity → Zustand
  └─ Want atomic state → Jotai
```

---

## Common Pitfalls

### ❌ **Pitfall 1: Using global state for everything**

```jsx
// BAD: Everything in global store
const useAppStore = create((set) => ({
  formValue: '',
  modalOpen: false,
  hoveredItem: null,
  // ... component-specific state doesn't belong here
}));

// GOOD: Use local state when possible
function MyForm() {
  const [formValue, setFormValue] = useState('');  // Local!
  // Only global state that's truly shared
}
```

---

### ❌ **Pitfall 2: Context performance issues**

```jsx
// BAD: Single context with all state
const AppContext = createContext();

function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  const [settings, setSettings] = useState({});
  
  // All consumers re-render when ANY value changes!
  return (
    <AppContext.Provider value={{ user, setUser, theme, setTheme, settings, setSettings }}>
      {children}
    </AppContext.Provider>
  );
}

// GOOD: Separate contexts for unrelated state
function App() {
  return (
    <UserProvider>
      <ThemeProvider>
        <SettingsProvider>
          <Page />
        </SettingsProvider>
      </ThemeProvider>
    </UserProvider>
  );
}
```

---

### ❌ **Pitfall 3: Not memoizing Context values**

```jsx
// BAD: New object on every render
function Provider({ children }) {
  const [user, setUser] = useState(null);
  
  return (
    <Context.Provider value={{ user, setUser }}>  {/* New object! */}
      {children}
    </Context.Provider>
  );
}

// GOOD: Memoize value
function Provider({ children }) {
  const [user, setUser] = useState(null);
  
  const value = useMemo(() => ({ user, setUser }), [user]);
  
  return (
    <Context.Provider value={value}>
      {children}
    </Context.Provider>
  );
}
```

---

## Quick Self-Check

✅ **You understand state management if you can:**

1. Explain when to use local state vs. global state
2. Implement Context API with proper memoization
3. Set up a Redux store with Redux Toolkit
4. Build a Zustand store with actions
5. Explain the difference between client state and server state
6. Choose the right state solution for different scenarios
7. Identify performance issues with Context
8. Use React Query for server state
9. Understand trade-offs of each solution
10. Migrate between state solutions

---

## Interview Questions to Practice

**Beginner:**
1. When should you use useState vs. Context?
2. What is prop drilling and how do you avoid it?
3. What are the downsides of Context API?

**Intermediate:**
4. Compare Redux, Zustand, and Jotai
5. When would you use React Query instead of Redux?
6. How do you prevent unnecessary re-renders with Context?
7. Explain Redux Toolkit's createSlice

**Advanced:**
8. How would you architect state for a large e-commerce app?
9. Explain the trade-offs of atomic vs. monolithic state
10. How would you migrate from Redux to Zustand?

---

## Further Practice

**Build these:**
1. **Todo app** with multiple state solutions (compare)
2. **E-commerce cart** with Zustand
3. **Dashboard** with React Query + Zustand
4. **Multi-step form** with state machine
5. **Real-time chat** with optimistic updates

**Resources:**
- [Redux Toolkit docs](https://redux-toolkit.js.org/)
- [Zustand docs](https://zustand-demo.pmnd.rs/)
- [Jotai docs](https://jotai.org/)
- [React Query docs](https://tanstack.com/query)

---

## Summary

**Local state (useState):**
- Use first, only lift when needed
- Best for: Component-specific state

**Context API:**
- Built-in, no dependencies
- Best for: Theme, auth, simple global state
- Watch out for: Performance with frequent updates

**Redux Toolkit:**
- Industry standard
- Best for: Large apps, complex state, teams
- Trade-off: More boilerplate

**Zustand:**
- Simple, modern
- Best for: Most apps, rapid development
- Trade-off: Smaller ecosystem

**Jotai:**
- Atomic, composable
- Best for: Derived state, fine-grained updates
- Trade-off: Different mental model

**React Query:**
- Server state specialist
- Best for: API data, caching
- Trade-off: Not for UI state

**Remember:** Start simple (useState), lift when needed, add global state only when truly global, and use React Query for server data.
