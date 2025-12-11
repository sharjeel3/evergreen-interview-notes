# Components & Props

**Level:** 🏁 Beginner  
**Tags:** `components`, `props`, `composition`, `interview`, `fundamentals`

---

## Why this matters

Components are the building blocks of React. In interviews, you must:

- explain the difference between function and class components
- understand one-way data flow via props
- know component composition patterns
- recognize props vs state

**Interview red flag:** modifying props directly

---

## Core Ideas

### 1. **Components are functions that return UI**

Modern React uses **function components** as the default:

```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

**Mental model:**  
"Components are UI functions: `data in → JSX out`"

---

### 2. **Props = read-only data passed from parent**

Props flow **downward** (parent → child), never upward.

```jsx
// Parent
function App() {
  return <Greeting name="Sarah" age={25} />;
}

// Child
function Greeting(props) {
  console.log(props);  // { name: "Sarah", age: 25 }
  return <h1>Hello, {props.name}</h1>;
}
```

**Key rule:** Props are **immutable**. Never do:
```jsx
props.name = "Different";  // ❌ DON'T!
```

---

### 3. **Destructuring props is idiomatic**

```jsx
// ❌ Repetitive
function Greeting(props) {
  return <h1>Hello, {props.name} ({props.age})</h1>;
}

// ✅ Clean and common
function Greeting({ name, age }) {
  return <h1>Hello, {name} ({age})</h1>;
}
```

---

### 4. **Default props and prop validation**

```jsx
// Default props (using default parameters)
function Button({ text = "Click me", onClick }) {
  return <button onClick={onClick}>{text}</button>;
}

// PropTypes (runtime validation, mostly legacy)
import PropTypes from 'prop-types';

Button.propTypes = {
  text: PropTypes.string,
  onClick: PropTypes.func.isRequired
};
```

**Modern approach:** Use TypeScript for compile-time type safety.

---

### 5. **Composition over inheritance**

React favors **composition** (building complex UIs from simple components) over inheritance.

```jsx
// ✅ Composition pattern
function Dialog({ title, children }) {
  return (
    <div className="dialog">
      <h1>{title}</h1>
      <div className="content">{children}</div>
    </div>
  );
}

function WelcomeDialog() {
  return (
    <Dialog title="Welcome">
      <p>Thank you for visiting!</p>
    </Dialog>
  );
}
```

**Mental model:**  
"Build with Lego blocks, not inheritance hierarchies."

---

## Examples

### Example 1 — Function vs class component

```jsx
// ✅ Modern: Function component
function Card({ title, description }) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <p>{description}</p>
    </div>
  );
}

// ⚠️ Legacy: Class component
class Card extends React.Component {
  render() {
    const { title, description } = this.props;
    return (
      <div className="card">
        <h2>{title}</h2>
        <p>{description}</p>
      </div>
    );
  }
}
```

**Note:** Class components are legacy. Use functions + hooks.

---

### Example 2 — Props with children

```jsx
function Container({ children }) {
  return <div className="container">{children}</div>;
}

// Usage
<Container>
  <h1>Title</h1>
  <p>Content here</p>
</Container>
```

`children` is a special prop containing nested content.

---

### Example 3 — Spreading props

```jsx
function Input(props) {
  return <input {...props} />;  // Forward all props
}

// Usage
<Input type="text" placeholder="Enter name" onChange={handleChange} />
```

Useful for wrapping native elements or HOCs.

---

### Example 4 — Conditional prop rendering

```jsx
function Button({ primary, children, onClick }) {
  return (
    <button 
      className={primary ? 'btn-primary' : 'btn-secondary'}
      onClick={onClick}
    >
      {children}
    </button>
  );
}

// Usage
<Button primary onClick={handleClick}>Save</Button>
<Button onClick={handleCancel}>Cancel</Button>
```

---

### Example 5 — Component composition

```jsx
function Avatar({ user }) {
  return <img src={user.avatarUrl} alt={user.name} />;
}

function UserInfo({ user }) {
  return (
    <div>
      <Avatar user={user} />
      <div>{user.name}</div>
    </div>
  );
}

function Comment({ author, text, date }) {
  return (
    <div className="comment">
      <UserInfo user={author} />
      <div className="text">{text}</div>
      <div className="date">{date}</div>
    </div>
  );
}
```

**Pattern:** Extract small, reusable components.

---

## Common Pitfalls

### 1. **Mutating props**

```jsx
// ❌ Props are read-only!
function BadComponent(props) {
  props.count += 1;  // ERROR: Don't modify props
  return <div>{props.count}</div>;
}

// ✅ Use state if you need to change data
function GoodComponent({ initialCount }) {
  const [count, setCount] = useState(initialCount);
  return <div>{count}</div>;
}
```

---

### 2. **Passing props incorrectly**

```jsx
// ❌ String without braces (always passes string)
<MyComponent count="5" />  // count is "5" (string), not 5 (number)

// ✅ Numbers, booleans, objects need braces
<MyComponent count={5} />
<MyComponent isActive={true} />
<MyComponent user={{ name: "Alice" }} />
```

---

### 3. **Key prop confusion**

```jsx
// ❌ Missing key in lists
{items.map(item => <ListItem item={item} />)}

// ✅ Always provide key for list items
{items.map(item => <ListItem key={item.id} item={item} />)}
```

**Note:** Key is internal to React, not passed to component.

---

### 4. **Overusing prop drilling**

```jsx
// ❌ Props passed through many layers
<GrandParent>
  <Parent user={user}>
    <Child user={user}>
      <GrandChild user={user} />
```

**Solutions:**
- Context API
- State management library (Redux, Zustand)
- Component composition

---

### 5. **Confusing props and state**

| Props | State |
|-------|-------|
| Passed from parent | Managed within component |
| Read-only | Can be updated |
| External data | Internal data |

---

## FAQ

**Q: Can a component change its own props?**  
A: No. Props are read-only. If a component needs to change data, use state.

**Q: What's the difference between `props.children` and other props?**  
A: `children` is a special prop for nested content. All other props are passed explicitly.

**Q: When should I use class components?**  
A: Almost never. Use function components + hooks. Classes are legacy.

**Q: How do I pass a callback to a child?**  
A: Pass a function as a prop:
```jsx
<Child onSave={handleSave} />
```

**Q: Can props be any data type?**  
A: Yes: strings, numbers, booleans, objects, arrays, functions, even React elements.

---

## Quick Self-Check

1. What's wrong with this code?
   ```jsx
   function Counter(props) {
     props.count++;
     return <div>{props.count}</div>;
   }
   ```

2. How do you pass a number prop?
   ```jsx
   <MyComponent value={42} />  // Not "42"!
   ```

3. What does `props.children` contain?

4. Why composition over inheritance in React?

5. How do you provide default props?
   ```jsx
   function Greeting({ name = "Guest" }) { ... }
   ```

---

## Further Practice

1. **Build:** Create a Card component with title, description, image, and button
2. **Refactor:** Take a large component and extract smaller reusable pieces
3. **Experiment:** Pass different data types as props (strings, numbers, functions, objects)
4. **Explore:** Use children prop to create a Layout component
5. **Debug:** Intentionally mutate props and observe React warnings

---

## Key Takeaways

- Components are functions that return JSX
- Props flow one-way (parent → child) and are read-only
- Destructure props for cleaner code
- Use `children` prop for component composition
- Favor composition over inheritance
- Never mutate props—use state for changes
- Class components are legacy; use functions + hooks
