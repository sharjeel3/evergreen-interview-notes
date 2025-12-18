# TypeScript with React

**Level:** 🔥 Advanced  
**Tags:** `typescript`, `types`, `generics`, `props`, `hooks`, `interview`

---

## Why this matters

TypeScript has become the industry standard for React development in 2024-2025. Most companies now require or strongly prefer TypeScript, and interviewers expect mid-to-senior developers to be proficient. Understanding TypeScript with React isn't just about adding types—it's about designing type-safe APIs, leveraging inference, and building maintainable systems.

**Why interviewers care:**
- TypeScript proficiency is now a baseline expectation at many companies
- Typing patterns reveal understanding of React's APIs and component contracts
- Generic components show advanced TypeScript knowledge
- Type safety discussions demonstrate concern for code quality and maintainability
- Interviewers test if you fight the type system or work with it
- Migration strategies (JS → TS) show practical experience

**Real-world implications:**
- **Catch bugs early:** Type errors caught at compile time, not runtime
- **Better IDE support:** Autocomplete, refactoring, inline documentation
- **Self-documenting code:** Types serve as documentation
- **Safer refactoring:** Compiler tells you what breaks when you change code
- **Team collaboration:** Types communicate component contracts clearly
- **Reduced testing burden:** Type system catches many bugs tests would otherwise catch
- **Hiring:** TypeScript skills make you more marketable

**Common TypeScript + React challenges:**
- Typing props (optional, required, children)
- Typing events (onClick, onChange, etc.)
- Typing hooks (useState, useRef, custom hooks)
- Generic components
- Typing Context
- Typing HOCs and render props
- Third-party library types
- Balancing type safety with developer experience

**What you must know:**
- Basic TypeScript (interfaces, types, generics, union types)
- Typing function components and props
- Typing hooks (useState, useRef, useReducer, useContext)
- Event types (React.ChangeEvent, React.MouseEvent, etc.)
- React built-in types (React.FC, React.ReactNode, React.ComponentProps)
- When to use `interface` vs. `type`
- Generic components

**Interview red flags:** Using `any` everywhere, fighting the type system, or claiming "TypeScript slows me down" without understanding the benefits.

---

## Core Ideas

### 1. **Typing Function Components and Props**

**Basic component:**

```tsx
// Props interface
interface ButtonProps {
  label: string;
  onClick: () => void;
  disabled?: boolean;  // Optional
}

// Function component
function Button({ label, onClick, disabled = false }: ButtonProps) {
  return (
    <button onClick={onClick} disabled={disabled}>
      {label}
    </button>
  );
}

// Usage
<Button label="Click me" onClick={() => console.log('clicked')} />
```

**With children:**

```tsx
interface CardProps {
  title: string;
  children: React.ReactNode;  // Accepts any valid JSX
}

function Card({ title, children }: CardProps) {
  return (
    <div>
      <h2>{title}</h2>
      {children}
    </div>
  );
}

// Usage
<Card title="My Card">
  <p>Card content</p>
</Card>
```

**React.ReactNode vs. React.ReactElement vs. JSX.Element:**

```tsx
// React.ReactNode - Most permissive (accepts strings, numbers, null, elements, etc.)
interface Props1 {
  children: React.ReactNode;  // ✅ Use this most of the time
}

// React.ReactElement - Must be a React element (not strings/numbers)
interface Props2 {
  children: React.ReactElement;
}

// JSX.Element - Single React element (more restrictive)
interface Props3 {
  children: JSX.Element;
}

// Examples
<Comp1>Hello</Comp1>  // ✅ ReactNode accepts strings
<Comp2>Hello</Comp2>  // ❌ ReactElement doesn't accept strings
<Comp2><span>Hi</span></Comp2>  // ✅ ReactElement accepts elements
```

**Optional vs. required props:**

```tsx
interface UserCardProps {
  name: string;           // Required
  email: string;          // Required
  avatar?: string;        // Optional
  age?: number;           // Optional
  onEdit?: () => void;    // Optional callback
}

function UserCard({ 
  name, 
  email, 
  avatar, 
  age,
  onEdit 
}: UserCardProps) {
  return (
    <div>
      {avatar && <img src={avatar} alt={name} />}
      <h3>{name}</h3>
      <p>{email}</p>
      {age && <p>Age: {age}</p>}
      {onEdit && <button onClick={onEdit}>Edit</button>}
    </div>
  );
}
```

**Default props:**

```tsx
interface ButtonProps {
  label: string;
  variant?: 'primary' | 'secondary' | 'danger';
}

function Button({ label, variant = 'primary' }: ButtonProps) {
  return <button className={variant}>{label}</button>;
}
```

---

### 2. **Typing Events**

**Common event types:**

```tsx
function Form() {
  // Input change
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    console.log(e.target.value);
  };

  // Textarea change
  const handleTextareaChange = (e: React.ChangeEvent<HTMLTextAreaElement>) => {
    console.log(e.target.value);
  };

  // Select change
  const handleSelectChange = (e: React.ChangeEvent<HTMLSelectElement>) => {
    console.log(e.target.value);
  };

  // Button click
  const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {
    console.log('Clicked at', e.clientX, e.clientY);
  };

  // Form submit
  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    console.log('Submitted');
  };

  // Key press
  const handleKeyPress = (e: React.KeyboardEvent<HTMLInputElement>) => {
    if (e.key === 'Enter') {
      console.log('Enter pressed');
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input onChange={handleChange} onKeyPress={handleKeyPress} />
      <textarea onChange={handleTextareaChange} />
      <select onChange={handleSelectChange}>
        <option>Option 1</option>
      </select>
      <button onClick={handleClick}>Submit</button>
    </form>
  );
}
```

**Generic event handler:**

```tsx
type InputChangeHandler = (e: React.ChangeEvent<HTMLInputElement>) => void;

interface FormProps {
  onEmailChange: InputChangeHandler;
  onPasswordChange: InputChangeHandler;
}

function LoginForm({ onEmailChange, onPasswordChange }: FormProps) {
  return (
    <div>
      <input type="email" onChange={onEmailChange} />
      <input type="password" onChange={onPasswordChange} />
    </div>
  );
}
```

---

### 3. **Typing Hooks**

**useState:**

```tsx
// Type is inferred
const [count, setCount] = useState(0);  // number
const [name, setName] = useState('');   // string

// Explicit type (when initial value is null/undefined)
const [user, setUser] = useState<User | null>(null);

interface User {
  id: string;
  name: string;
  email: string;
}

function UserProfile() {
  const [user, setUser] = useState<User | null>(null);

  useEffect(() => {
    fetchUser().then(setUser);
  }, []);

  if (!user) return <div>Loading...</div>;

  return <div>{user.name}</div>;  // ✅ TypeScript knows user is not null
}
```

**useRef:**

```tsx
// DOM ref
function TextInput() {
  const inputRef = useRef<HTMLInputElement>(null);

  const focusInput = () => {
    inputRef.current?.focus();  // Optional chaining (ref might be null)
  };

  return (
    <>
      <input ref={inputRef} />
      <button onClick={focusInput}>Focus</button>
    </>
  );
}

// Mutable value ref
function Timer() {
  const intervalRef = useRef<number | null>(null);

  const startTimer = () => {
    intervalRef.current = window.setInterval(() => {
      console.log('Tick');
    }, 1000);
  };

  const stopTimer = () => {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
    }
  };

  return (
    <>
      <button onClick={startTimer}>Start</button>
      <button onClick={stopTimer}>Stop</button>
    </>
  );
}
```

**useReducer:**

```tsx
// State type
interface State {
  count: number;
  error: string | null;
}

// Action types (discriminated union)
type Action =
  | { type: 'increment' }
  | { type: 'decrement' }
  | { type: 'set'; payload: number }
  | { type: 'error'; payload: string };

// Reducer
function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'increment':
      return { ...state, count: state.count + 1 };
    case 'decrement':
      return { ...state, count: state.count - 1 };
    case 'set':
      return { ...state, count: action.payload };  // ✅ payload is number
    case 'error':
      return { ...state, error: action.payload };  // ✅ payload is string
    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0, error: null });

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'set', payload: 10 })}>Set 10</button>
    </div>
  );
}
```

**useContext:**

```tsx
interface User {
  id: string;
  name: string;
}

interface UserContextType {
  user: User | null;
  login: (user: User) => void;
  logout: () => void;
}

// Create context with type
const UserContext = createContext<UserContextType | undefined>(undefined);

// Provider
function UserProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);

  const login = (user: User) => setUser(user);
  const logout = () => setUser(null);

  return (
    <UserContext.Provider value={{ user, login, logout }}>
      {children}
    </UserContext.Provider>
  );
}

// Custom hook with type safety
function useUser() {
  const context = useContext(UserContext);
  if (context === undefined) {
    throw new Error('useUser must be used within UserProvider');
  }
  return context;
}

// Usage
function Profile() {
  const { user, logout } = useUser();  // ✅ Fully typed

  return (
    <div>
      <p>{user?.name}</p>
      <button onClick={logout}>Logout</button>
    </div>
  );
}
```

**Custom hooks:**

```tsx
// Fetch hook with generics
function useFetch<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then((data: T) => {
        setData(data);
        setLoading(false);
      })
      .catch((err: Error) => {
        setError(err);
        setLoading(false);
      });
  }, [url]);

  return { data, loading, error };
}

// Usage
interface User {
  id: string;
  name: string;
}

function UserList() {
  const { data, loading, error } = useFetch<User[]>('/api/users');

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  
  return (
    <ul>
      {data?.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

---

### 4. **Generic Components**

**List component:**

```tsx
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
  keyExtractor: (item: T) => string | number;
}

function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  return (
    <ul>
      {items.map(item => (
        <li key={keyExtractor(item)}>
          {renderItem(item)}
        </li>
      ))}
    </ul>
  );
}

// Usage
interface User {
  id: string;
  name: string;
  email: string;
}

function UserList() {
  const users: User[] = [
    { id: '1', name: 'Alice', email: 'alice@example.com' },
    { id: '2', name: 'Bob', email: 'bob@example.com' }
  ];

  return (
    <List
      items={users}
      keyExtractor={user => user.id}
      renderItem={user => (
        <div>
          {user.name} - {user.email}
        </div>
      )}
    />
  );
}
```

**Select component:**

```tsx
interface SelectProps<T> {
  options: T[];
  value: T;
  onChange: (value: T) => void;
  getLabel: (option: T) => string;
  getValue: (option: T) => string;
}

function Select<T>({ options, value, onChange, getLabel, getValue }: SelectProps<T>) {
  const handleChange = (e: React.ChangeEvent<HTMLSelectElement>) => {
    const selectedValue = e.target.value;
    const option = options.find(opt => getValue(opt) === selectedValue);
    if (option) onChange(option);
  };

  return (
    <select value={getValue(value)} onChange={handleChange}>
      {options.map(option => (
        <option key={getValue(option)} value={getValue(option)}>
          {getLabel(option)}
        </option>
      ))}
    </select>
  );
}

// Usage
interface Country {
  code: string;
  name: string;
}

function CountrySelector() {
  const countries: Country[] = [
    { code: 'US', name: 'United States' },
    { code: 'CA', name: 'Canada' }
  ];

  const [selected, setSelected] = useState(countries[0]);

  return (
    <Select
      options={countries}
      value={selected}
      onChange={setSelected}
      getLabel={c => c.name}
      getValue={c => c.code}
    />
  );
}
```

---

### 5. **Advanced Patterns**

**Discriminated unions for props:**

```tsx
// Button can be either a link or a button
type ButtonProps =
  | { as: 'button'; onClick: () => void; type?: 'submit' | 'button' }
  | { as: 'link'; href: string; target?: string };

function Button(props: ButtonProps & { children: React.ReactNode }) {
  if (props.as === 'link') {
    return <a href={props.href} target={props.target}>{props.children}</a>;
  }

  return (
    <button onClick={props.onClick} type={props.type}>
      {props.children}
    </button>
  );
}

// Usage
<Button as="button" onClick={() => {}}>Click</Button>
<Button as="link" href="/home">Go Home</Button>
```

**Extending HTML attributes:**

```tsx
// Extend button props with custom props
interface CustomButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary';
  isLoading?: boolean;
}

function CustomButton({ 
  variant = 'primary', 
  isLoading, 
  children, 
  disabled,
  ...rest  // All other HTML button props
}: CustomButtonProps) {
  return (
    <button 
      className={variant} 
      disabled={disabled || isLoading}
      {...rest}  // onClick, type, etc.
    >
      {isLoading ? 'Loading...' : children}
    </button>
  );
}

// Usage - All button props work!
<CustomButton 
  variant="primary" 
  onClick={() => {}} 
  type="submit"
  aria-label="Submit form"
>
  Submit
</CustomButton>
```

**Component props utility:**

```tsx
// Extract props from an existing component
type ButtonProps = React.ComponentProps<typeof Button>;

// Or from an HTML element
type DivProps = React.ComponentProps<'div'>;
type InputProps = React.ComponentProps<'input'>;
```

---

## Common Pitfalls

### ❌ **Pitfall 1: Overusing `any`**

```tsx
// BAD: Defeats the purpose of TypeScript
function processData(data: any) {
  return data.map((item: any) => item.value);  // No type safety!
}

// GOOD: Use proper types or generics
function processData<T extends { value: string }>(data: T[]) {
  return data.map(item => item.value);
}
```

---

### ❌ **Pitfall 2: Using React.FC (deprecated pattern)**

```tsx
// AVOID: React.FC has issues (implicit children, not recommended)
const Button: React.FC<{ label: string }> = ({ label }) => {
  return <button>{label}</button>;
};

// PREFER: Explicit function component
interface ButtonProps {
  label: string;
  children?: React.ReactNode;  // Explicit
}

function Button({ label, children }: ButtonProps) {
  return <button>{label}{children}</button>;
}
```

---

### ❌ **Pitfall 3: Not using discriminated unions**

```tsx
// BAD: Runtime error possible
interface ApiResponse {
  status: 'success' | 'error';
  data?: User;
  error?: string;
}

function Component({ response }: { response: ApiResponse }) {
  if (response.status === 'success') {
    return <div>{response.data.name}</div>;  // ❌ data might be undefined!
  }
  return <div>{response.error}</div>;  // ❌ error might be undefined!
}

// GOOD: Discriminated union
type ApiResponse =
  | { status: 'success'; data: User }
  | { status: 'error'; error: string };

function Component({ response }: { response: ApiResponse }) {
  if (response.status === 'success') {
    return <div>{response.data.name}</div>;  // ✅ TypeScript knows data exists
  }
  return <div>{response.error}</div>;  // ✅ TypeScript knows error exists
}
```

---

## Quick Self-Check

✅ **You understand TypeScript with React if you can:**

1. Type function component props (required, optional, children)
2. Type event handlers (onChange, onClick, onSubmit)
3. Type hooks (useState, useRef, useReducer, useContext)
4. Create a generic component (List, Select)
5. Use discriminated unions for conditional props
6. Extend HTML element props
7. Type custom hooks
8. Explain when to use `interface` vs. `type`
9. Type Context API properly
10. Avoid common pitfalls (any, React.FC)

---

## Interview Questions to Practice

**Beginner:**
1. How do you type props for a React component?
2. What's the difference between `React.ReactNode` and `React.ReactElement`?
3. How do you type an onClick handler?

**Intermediate:**
4. How do you type a useState hook with an object?
5. How do you create a generic List component?
6. What's a discriminated union and when would you use it?
7. How do you extend HTML button props with custom props?

**Advanced:**
8. How would you type a custom hook that returns different types based on a parameter?
9. Explain how to type the Context API with type safety
10. How would you migrate a large JavaScript React codebase to TypeScript?

---

## Further Practice

**Build and type these:**
1. **Generic Table component** with sortable columns
2. **Form library** with type-safe field registration
3. **Data fetching hook** with loading/error/success states
4. **Multi-step wizard** with type-safe step data
5. **Component library** extending HTML elements (Button, Input, etc.)

**Resources:**
- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- [DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped) - Types for libraries

---

## Summary

**Core patterns:**
- Function components with typed props
- Event types (React.ChangeEvent, React.MouseEvent, etc.)
- Hook types (useState, useRef, useReducer, useContext)
- Generic components with type parameters
- Discriminated unions for conditional props
- Extending HTML attributes

**Best practices:**
- Avoid `any` - use `unknown` if truly unknown
- Don't use `React.FC` (deprecated)
- Use discriminated unions for related but different states
- Leverage TypeScript's inference (don't over-annotate)
- Use generics for reusable components

**Remember:** TypeScript is a tool to help you, not fight you. If you're fighting the type system, you might be modeling your data incorrectly.
