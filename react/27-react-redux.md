# React-Redux Integration

**Level:** 🔥 Advanced  
**Tags:** `react-redux`, `useSelector`, `useDispatch`, `redux-toolkit`, `state-management`, `hooks`, `connect`

---

## Why this matters

React-Redux is the official React binding for Redux, used by thousands of production applications. Interviewers use React-Redux questions to assess your ability to integrate Redux into React applications, optimize performance, and follow best practices. While newer alternatives exist (Zustand, Jotai), React-Redux remains the most common state management solution in enterprise applications, making it essential interview knowledge.

**Why interviewers care:**
- React-Redux is the industry standard (most legacy and current codebases use it)
- Tests understanding of React performance optimization (re-render prevention)
- Shows knowledge of modern hooks API vs. legacy connect() HOC
- Demonstrates ability to structure large-scale applications
- Senior roles expect experience with Redux middleware and DevTools
- Migration questions assess architectural decision-making skills

**Real-world implications:**
- **Enterprise apps:** Most large companies use React-Redux in production
- **Performance:** Poor Redux integration causes unnecessary re-renders
- **Maintainability:** Proper patterns make large codebases manageable
- **Team collaboration:** Shared Redux patterns improve team productivity
- **Testing:** Well-structured Redux code is easier to test
- **Debugging:** Redux DevTools make debugging state changes trivial

**Common React-Redux challenges:**
- Selecting state efficiently (avoiding unnecessary re-renders)
- Dispatching actions from components
- Typing Redux with TypeScript
- Testing connected components
- Migrating from connect() to hooks
- Optimizing selector performance
- Handling async actions

**What you must know:**
- useSelector hook (read Redux state)
- useDispatch hook (dispatch actions)
- Redux Toolkit integration (modern approach)
- Selector patterns and memoization
- connect() HOC (legacy but still used)
- Performance optimization techniques
- TypeScript patterns
- Testing strategies

**Interview red flags:** Only knowing connect(), not understanding useSelector optimization, creating selectors inline, or not knowing about reselect/memoization.

---

## Core Ideas

### 1. **Setting Up Redux with React**

**Install dependencies:**

```bash
npm install @reduxjs/toolkit react-redux
```

**Store setup with Redux Toolkit:**

```tsx
// store/index.ts
import { configureStore } from '@reduxjs/toolkit';
import userReducer from './userSlice';
import cartReducer from './cartSlice';

export const store = configureStore({
  reducer: {
    user: userReducer,
    cart: cartReducer,
  },
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

**Provide store to React app:**

```tsx
// index.tsx or App.tsx
import { Provider } from 'react-redux';
import { store } from './store';

function App() {
  return (
    <Provider store={store}>
      <YourApp />
    </Provider>
  );
}

export default App;
```

**Create a slice (Redux Toolkit):**

```tsx
// store/userSlice.ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface User {
  id: string;
  name: string;
  email: string;
}

interface UserState {
  currentUser: User | null;
  isLoading: boolean;
  error: string | null;
}

const initialState: UserState = {
  currentUser: null,
  isLoading: false,
  error: null,
};

const userSlice = createSlice({
  name: 'user',
  initialState,
  reducers: {
    setUser: (state, action: PayloadAction<User>) => {
      state.currentUser = action.payload;
      state.isLoading = false;
      state.error = null;
    },
    clearUser: (state) => {
      state.currentUser = null;
    },
    setLoading: (state, action: PayloadAction<boolean>) => {
      state.isLoading = action.payload;
    },
    setError: (state, action: PayloadAction<string>) => {
      state.error = action.payload;
      state.isLoading = false;
    },
  },
});

export const { setUser, clearUser, setLoading, setError } = userSlice.actions;
export default userSlice.reducer;
```

---

### 2. **useSelector: Reading Redux State**

**Basic useSelector:**

```tsx
import { useSelector } from 'react-redux';
import { RootState } from './store';

function UserProfile() {
  // Select user from Redux state
  const user = useSelector((state: RootState) => state.user.currentUser);
  
  if (!user) {
    return <div>Please login</div>;
  }
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

**Selecting multiple values:**

```tsx
function Dashboard() {
  const user = useSelector((state: RootState) => state.user.currentUser);
  const isLoading = useSelector((state: RootState) => state.user.isLoading);
  const cartItems = useSelector((state: RootState) => state.cart.items);
  
  if (isLoading) {
    return <div>Loading...</div>;
  }
  
  return (
    <div>
      <h1>Welcome, {user?.name}</h1>
      <p>Cart items: {cartItems.length}</p>
    </div>
  );
}
```

**Typed hooks (recommended pattern):**

```tsx
// store/hooks.ts
import { useDispatch, useSelector } from 'react-redux';
import type { RootState, AppDispatch } from './index';

// Pre-typed hooks
export const useAppDispatch = useDispatch.withTypes<AppDispatch>();
export const useAppSelector = useSelector.withTypes<RootState>();
```

```tsx
// Usage in components
import { useAppSelector, useAppDispatch } from '@/store/hooks';

function UserProfile() {
  const user = useAppSelector(state => state.user.currentUser);
  // TypeScript knows state type automatically!
}
```

---

### 3. **useDispatch: Dispatching Actions**

**Basic dispatch:**

```tsx
import { useDispatch } from 'react-redux';
import { setUser, clearUser } from '@/store/userSlice';

function LoginForm() {
  const dispatch = useDispatch();
  
  const handleLogin = async (email: string, password: string) => {
    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        body: JSON.stringify({ email, password }),
      });
      const user = await response.json();
      
      dispatch(setUser(user));
    } catch (error) {
      console.error('Login failed', error);
    }
  };
  
  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      handleLogin('user@example.com', 'password');
    }}>
      <button type="submit">Login</button>
    </form>
  );
}

function LogoutButton() {
  const dispatch = useDispatch();
  
  return (
    <button onClick={() => dispatch(clearUser())}>
      Logout
    </button>
  );
}
```

**Typed dispatch:**

```tsx
import { useAppDispatch } from '@/store/hooks';

function Component() {
  const dispatch = useAppDispatch();  // Typed!
  
  dispatch(setUser({ id: '1', name: 'John', email: 'john@example.com' }));
  // TypeScript validates action payload
}
```

---

### 4. **Async Actions with createAsyncThunk**

**Define async thunk:**

```tsx
// store/userSlice.ts
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

export const fetchUser = createAsyncThunk(
  'user/fetchUser',
  async (userId: string, { rejectWithValue }) => {
    try {
      const response = await fetch(`/api/users/${userId}`);
      if (!response.ok) {
        throw new Error('Failed to fetch user');
      }
      return await response.json();
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

export const loginUser = createAsyncThunk(
  'user/login',
  async ({ email, password }: { email: string; password: string }) => {
    const response = await fetch('/api/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email, password }),
    });
    return response.json();
  }
);

const userSlice = createSlice({
  name: 'user',
  initialState,
  reducers: {
    clearUser: (state) => {
      state.currentUser = null;
    },
  },
  extraReducers: (builder) => {
    builder
      // fetchUser
      .addCase(fetchUser.pending, (state) => {
        state.isLoading = true;
        state.error = null;
      })
      .addCase(fetchUser.fulfilled, (state, action) => {
        state.currentUser = action.payload;
        state.isLoading = false;
      })
      .addCase(fetchUser.rejected, (state, action) => {
        state.isLoading = false;
        state.error = action.payload as string;
      })
      // loginUser
      .addCase(loginUser.pending, (state) => {
        state.isLoading = true;
      })
      .addCase(loginUser.fulfilled, (state, action) => {
        state.currentUser = action.payload;
        state.isLoading = false;
      })
      .addCase(loginUser.rejected, (state, action) => {
        state.isLoading = false;
        state.error = action.error.message || 'Login failed';
      });
  },
});
```

**Dispatch async thunk in component:**

```tsx
import { useEffect } from 'react';
import { useAppDispatch, useAppSelector } from '@/store/hooks';
import { fetchUser } from '@/store/userSlice';

function UserProfile({ userId }: { userId: string }) {
  const dispatch = useAppDispatch();
  const { currentUser, isLoading, error } = useAppSelector(state => state.user);
  
  useEffect(() => {
    dispatch(fetchUser(userId));
  }, [dispatch, userId]);
  
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!currentUser) return <div>No user found</div>;
  
  return (
    <div>
      <h1>{currentUser.name}</h1>
      <p>{currentUser.email}</p>
    </div>
  );
}
```

**Login form with async thunk:**

```tsx
import { useState } from 'react';
import { useAppDispatch, useAppSelector } from '@/store/hooks';
import { loginUser } from '@/store/userSlice';

function LoginForm() {
  const dispatch = useAppDispatch();
  const { isLoading, error } = useAppSelector(state => state.user);
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    const result = await dispatch(loginUser({ email, password }));
    
    if (loginUser.fulfilled.match(result)) {
      console.log('Login successful!');
    } else {
      console.error('Login failed:', result.error);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Password"
      />
      <button type="submit" disabled={isLoading}>
        {isLoading ? 'Logging in...' : 'Login'}
      </button>
      {error && <div className="error">{error}</div>}
    </form>
  );
}
```

---

### 5. **Selector Performance and Memoization**

**❌ Problem: Inline selectors cause re-renders**

```tsx
// BAD: New selector function on every render
function TodoList() {
  const completedTodos = useSelector((state: RootState) => 
    state.todos.items.filter(todo => todo.completed)
  );  // New array every time → re-renders even if no todos changed!
  
  return <div>{completedTodos.length} completed</div>;
}
```

**✅ Solution 1: Extract selector**

```tsx
// store/todoSelectors.ts
import { RootState } from './index';

export const selectCompletedTodos = (state: RootState) =>
  state.todos.items.filter(todo => todo.completed);

// Component
function TodoList() {
  const completedTodos = useAppSelector(selectCompletedTodos);
  // Still creates new array, but at least selector is stable
}
```

**✅ Solution 2: Reselect (memoized selectors)**

```bash
npm install reselect
```

```tsx
// store/todoSelectors.ts
import { createSelector } from '@reduxjs/toolkit';
import { RootState } from './index';

// Input selector
const selectTodos = (state: RootState) => state.todos.items;

// Memoized selector (only recalculates when todos change)
export const selectCompletedTodos = createSelector(
  [selectTodos],
  (todos) => todos.filter(todo => todo.completed)
);

export const selectActiveTodos = createSelector(
  [selectTodos],
  (todos) => todos.filter(todo => !todo.completed)
);

// Parameterized selector
export const selectTodoById = createSelector(
  [selectTodos, (state: RootState, id: string) => id],
  (todos, id) => todos.find(todo => todo.id === id)
);
```

```tsx
// Component
import { selectCompletedTodos, selectTodoById } from '@/store/todoSelectors';

function TodoList() {
  const completedTodos = useAppSelector(selectCompletedTodos);
  // Only re-renders if completed todos actually change
  
  return <div>{completedTodos.length} completed</div>;
}

function TodoItem({ todoId }: { todoId: string }) {
  const todo = useAppSelector(state => selectTodoById(state, todoId));
  
  return <div>{todo?.title}</div>;
}
```

**✅ Solution 3: Shallow equality check**

```tsx
import { shallowEqual } from 'react-redux';

function UserProfile() {
  const { name, email } = useAppSelector(
    (state: RootState) => ({
      name: state.user.currentUser?.name,
      email: state.user.currentUser?.email,
    }),
    shallowEqual  // Compare object properties, not reference
  );
  
  return <div>{name} - {email}</div>;
}
```

---

### 6. **connect() HOC (Legacy Pattern)**

**Basic connect:**

```tsx
import { connect } from 'react-redux';
import { RootState } from '@/store';
import { setUser, clearUser } from '@/store/userSlice';

interface OwnProps {
  userId: string;
}

interface StateProps {
  user: User | null;
  isLoading: boolean;
}

interface DispatchProps {
  setUser: typeof setUser;
  clearUser: typeof clearUser;
}

type Props = OwnProps & StateProps & DispatchProps;

class UserProfile extends React.Component<Props> {
  componentDidMount() {
    // Use this.props.setUser, this.props.user, etc.
  }
  
  render() {
    const { user, isLoading, clearUser } = this.props;
    
    if (isLoading) return <div>Loading...</div>;
    if (!user) return <div>No user</div>;
    
    return (
      <div>
        <h1>{user.name}</h1>
        <button onClick={() => clearUser()}>Logout</button>
      </div>
    );
  }
}

// mapStateToProps
const mapStateToProps = (state: RootState, ownProps: OwnProps): StateProps => ({
  user: state.user.currentUser,
  isLoading: state.user.isLoading,
});

// mapDispatchToProps
const mapDispatchToProps: DispatchProps = {
  setUser,
  clearUser,
};

export default connect(mapStateToProps, mapDispatchToProps)(UserProfile);
```

**Functional component with connect:**

```tsx
import { connect } from 'react-redux';

interface Props {
  user: User | null;
  clearUser: () => void;
}

function UserProfile({ user, clearUser }: Props) {
  if (!user) return <div>No user</div>;
  
  return (
    <div>
      <h1>{user.name}</h1>
      <button onClick={clearUser}>Logout</button>
    </div>
  );
}

export default connect(
  (state: RootState) => ({
    user: state.user.currentUser,
  }),
  { clearUser }
)(UserProfile);
```

**Why connect() is legacy:**
- Verbose (lots of boilerplate)
- Harder to type with TypeScript
- HOC pattern is less intuitive than hooks
- Can't use inside conditional or loops
- Hooks (useSelector/useDispatch) are simpler

**When you still see connect():**
- Legacy codebases
- Class components
- Some libraries still use it

**Migration from connect() to hooks:**

```tsx
// OLD: connect()
const mapStateToProps = (state: RootState) => ({
  user: state.user.currentUser,
  isLoading: state.user.isLoading,
});

const mapDispatchToProps = {
  setUser,
  clearUser,
};

export default connect(mapStateToProps, mapDispatchToProps)(UserProfile);

// NEW: hooks
function UserProfile() {
  const user = useAppSelector(state => state.user.currentUser);
  const isLoading = useAppSelector(state => state.user.isLoading);
  const dispatch = useAppDispatch();
  
  return (
    <div>
      <button onClick={() => dispatch(setUser(newUser))}>Set User</button>
      <button onClick={() => dispatch(clearUser())}>Clear User</button>
    </div>
  );
}
```

---

### 7. **Real-World Example: Shopping Cart**

**Cart slice:**

```tsx
// store/cartSlice.ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

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

const initialState: CartState = {
  items: [],
  total: 0,
};

const cartSlice = createSlice({
  name: 'cart',
  initialState,
  reducers: {
    addItem: (state, action: PayloadAction<Omit<CartItem, 'quantity'>>) => {
      const existingItem = state.items.find(item => item.id === action.payload.id);
      
      if (existingItem) {
        existingItem.quantity += 1;
      } else {
        state.items.push({ ...action.payload, quantity: 1 });
      }
      
      state.total = state.items.reduce(
        (sum, item) => sum + item.price * item.quantity,
        0
      );
    },
    removeItem: (state, action: PayloadAction<string>) => {
      state.items = state.items.filter(item => item.id !== action.payload);
      state.total = state.items.reduce(
        (sum, item) => sum + item.price * item.quantity,
        0
      );
    },
    updateQuantity: (state, action: PayloadAction<{ id: string; quantity: number }>) => {
      const item = state.items.find(item => item.id === action.payload.id);
      if (item) {
        item.quantity = action.payload.quantity;
        state.total = state.items.reduce(
          (sum, item) => sum + item.price * item.quantity,
          0
        );
      }
    },
    clearCart: (state) => {
      state.items = [];
      state.total = 0;
    },
  },
});

export const { addItem, removeItem, updateQuantity, clearCart } = cartSlice.actions;
export default cartSlice.reducer;
```

**Selectors:**

```tsx
// store/cartSelectors.ts
import { createSelector } from '@reduxjs/toolkit';
import { RootState } from './index';

export const selectCartItems = (state: RootState) => state.cart.items;
export const selectCartTotal = (state: RootState) => state.cart.total;

export const selectCartItemCount = createSelector(
  [selectCartItems],
  (items) => items.reduce((sum, item) => sum + item.quantity, 0)
);

export const selectCartItemById = createSelector(
  [selectCartItems, (state: RootState, id: string) => id],
  (items, id) => items.find(item => item.id === id)
);
```

**Components:**

```tsx
// components/ProductCard.tsx
import { useAppDispatch } from '@/store/hooks';
import { addItem } from '@/store/cartSlice';

interface Product {
  id: string;
  name: string;
  price: number;
}

export function ProductCard({ product }: { product: Product }) {
  const dispatch = useAppDispatch();
  
  const handleAddToCart = () => {
    dispatch(addItem({
      id: product.id,
      name: product.name,
      price: product.price,
    }));
  };
  
  return (
    <div className="product-card">
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      <button onClick={handleAddToCart}>Add to Cart</button>
    </div>
  );
}
```

```tsx
// components/CartSummary.tsx
import { useAppSelector } from '@/store/hooks';
import { selectCartItemCount, selectCartTotal } from '@/store/cartSelectors';

export function CartSummary() {
  const itemCount = useAppSelector(selectCartItemCount);
  const total = useAppSelector(selectCartTotal);
  
  return (
    <div className="cart-summary">
      <p>Items: {itemCount}</p>
      <p>Total: ${total.toFixed(2)}</p>
    </div>
  );
}
```

```tsx
// components/Cart.tsx
import { useAppSelector, useAppDispatch } from '@/store/hooks';
import { selectCartItems } from '@/store/cartSelectors';
import { removeItem, updateQuantity, clearCart } from '@/store/cartSlice';

export function Cart() {
  const items = useAppSelector(selectCartItems);
  const dispatch = useAppDispatch();
  
  return (
    <div className="cart">
      <h2>Shopping Cart</h2>
      
      {items.length === 0 ? (
        <p>Your cart is empty</p>
      ) : (
        <>
          {items.map(item => (
            <div key={item.id} className="cart-item">
              <h3>{item.name}</h3>
              <p>${item.price}</p>
              <input
                type="number"
                value={item.quantity}
                onChange={(e) => 
                  dispatch(updateQuantity({
                    id: item.id,
                    quantity: parseInt(e.target.value)
                  }))
                }
                min="1"
              />
              <button onClick={() => dispatch(removeItem(item.id))}>
                Remove
              </button>
            </div>
          ))}
          
          <button onClick={() => dispatch(clearCart())}>
            Clear Cart
          </button>
        </>
      )}
    </div>
  );
}
```

---

### 8. **Testing Redux-Connected Components**

**Setup test store:**

```tsx
// test-utils.tsx
import { configureStore } from '@reduxjs/toolkit';
import { Provider } from 'react-redux';
import { render } from '@testing-library/react';
import userReducer from '@/store/userSlice';
import cartReducer from '@/store/cartSlice';

export function renderWithRedux(
  ui: React.ReactElement,
  {
    preloadedState = {},
    store = configureStore({
      reducer: {
        user: userReducer,
        cart: cartReducer,
      },
      preloadedState,
    }),
    ...renderOptions
  } = {}
) {
  function Wrapper({ children }: { children: React.ReactNode }) {
    return <Provider store={store}>{children}</Provider>;
  }
  
  return { store, ...render(ui, { wrapper: Wrapper, ...renderOptions }) };
}
```

**Test component:**

```tsx
// components/UserProfile.test.tsx
import { screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { renderWithRedux } from '@/test-utils';
import { UserProfile } from './UserProfile';

describe('UserProfile', () => {
  it('displays user name', () => {
    renderWithRedux(<UserProfile />, {
      preloadedState: {
        user: {
          currentUser: { id: '1', name: 'John Doe', email: 'john@example.com' },
          isLoading: false,
          error: null,
        },
      },
    });
    
    expect(screen.getByText('John Doe')).toBeInTheDocument();
  });
  
  it('shows loading state', () => {
    renderWithRedux(<UserProfile />, {
      preloadedState: {
        user: {
          currentUser: null,
          isLoading: true,
          error: null,
        },
      },
    });
    
    expect(screen.getByText('Loading...')).toBeInTheDocument();
  });
  
  it('dispatches logout action', async () => {
    const { store } = renderWithRedux(<UserProfile />, {
      preloadedState: {
        user: {
          currentUser: { id: '1', name: 'John', email: 'john@example.com' },
          isLoading: false,
          error: null,
        },
      },
    });
    
    const logoutButton = screen.getByRole('button', { name: /logout/i });
    await userEvent.click(logoutButton);
    
    expect(store.getState().user.currentUser).toBeNull();
  });
});
```

---

## Common Pitfalls

### ❌ **Pitfall 1: Inline selectors with derived state**

```tsx
// BAD: Re-renders unnecessarily (new array every time)
function TodoList() {
  const completedTodos = useAppSelector(state =>
    state.todos.items.filter(todo => todo.completed)
  );
}

// GOOD: Memoized selector
const selectCompletedTodos = createSelector(
  [state => state.todos.items],
  items => items.filter(todo => todo.completed)
);

function TodoList() {
  const completedTodos = useAppSelector(selectCompletedTodos);
}
```

---

### ❌ **Pitfall 2: Not memoizing dispatch callbacks**

```tsx
// BAD: New function on every render
function TodoItem({ id }: { id: string }) {
  const dispatch = useAppDispatch();
  
  return (
    <ChildComponent 
      onDelete={() => dispatch(deleteTodo(id))}  // New function!
    />
  );
}

// GOOD: Memoize callback
function TodoItem({ id }: { id: string }) {
  const dispatch = useAppDispatch();
  
  const handleDelete = useCallback(() => {
    dispatch(deleteTodo(id));
  }, [dispatch, id]);
  
  return <ChildComponent onDelete={handleDelete} />;
}
```

---

### ❌ **Pitfall 3: Selecting entire state object**

```tsx
// BAD: Re-renders when ANY state changes
function Component() {
  const state = useAppSelector(state => state);  // Entire state!
  return <div>{state.user.currentUser?.name}</div>;
}

// GOOD: Select only what you need
function Component() {
  const userName = useAppSelector(state => state.user.currentUser?.name);
  return <div>{userName}</div>;
}
```

---

## Quick Self-Check

✅ **You understand React-Redux if you can:**

1. Set up Redux store and Provider
2. Use useSelector to read state
3. Use useDispatch to dispatch actions
4. Create async thunks with createAsyncThunk
5. Write memoized selectors with createSelector
6. Optimize component re-renders
7. Type Redux with TypeScript
8. Test Redux-connected components
9. Understand connect() HOC (legacy)
10. Migrate from connect() to hooks

---

## Interview Questions to Practice

**Beginner:**
1. How do you connect a React component to Redux?
2. What's the difference between useSelector and useDispatch?
3. How do you dispatch an action from a component?

**Intermediate:**
4. Why should you avoid inline selectors for derived state?
5. How do you handle async actions in Redux?
6. What is createSelector and when should you use it?
7. How do you type Redux with TypeScript?

**Advanced:**
8. How would you optimize Redux performance in a large app?
9. Explain the difference between connect() and hooks
10. How do you test components connected to Redux?

---

## Further Practice

**Build these:**
1. **Todo app** with Redux (CRUD operations)
2. **Shopping cart** (add/remove items, quantities)
3. **Authentication flow** (login, logout, protected routes)
4. **Real-time dashboard** (async data fetching)
5. **Multi-step form** (wizard with Redux state)

**Optimize:**
- Measure re-renders with React DevTools Profiler
- Compare inline selectors vs. memoized selectors
- Test different selector patterns

**Resources:**
- [React-Redux Docs](https://react-redux.js.org/)
- [Redux Toolkit Docs](https://redux-toolkit.js.org/)
- [Reselect Docs](https://github.com/reduxjs/reselect)

---

## Summary

**React-Redux integration:**
- **Provider:** Wraps app with Redux store
- **useSelector:** Read state (with memoization)
- **useDispatch:** Dispatch actions
- **createAsyncThunk:** Handle async operations
- **createSelector:** Memoized derived state

**Best practices:**
- Use typed hooks (useAppSelector, useAppDispatch)
- Memoize selectors for derived state
- Select only needed state (avoid re-renders)
- Use Redux Toolkit (less boilerplate)
- Test with custom renderWithRedux

**Performance optimization:**
- Avoid inline selectors with transformations
- Use createSelector for expensive computations
- Use shallowEqual for object selections
- Memoize callbacks passed to children

**Migration path:**
- Legacy: connect() HOC
- Modern: useSelector + useDispatch hooks
- Prefer hooks for new code
- Gradually migrate connect() to hooks

**Remember:** Start with hooks (useSelector/useDispatch), only use connect() for legacy code. Always memoize derived state with createSelector to prevent unnecessary re-renders.
