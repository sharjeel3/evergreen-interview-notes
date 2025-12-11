# JSX & Rendering

**Level:** 🏁 Beginner  
**Tags:** `jsx`, `virtual-dom`, `rendering`, `interview`, `fundamentals`

---

## Why this matters

JSX is React's signature syntax. In interviews, you need to:

- explain what JSX actually *is* (not HTML!)
- understand how it transforms to JavaScript
- know why React uses a Virtual DOM
- debug common JSX syntax errors

**Interview red flag:** saying "JSX is HTML in JavaScript"

---

## Core Ideas

### 1. **JSX is syntactic sugar for function calls**

JSX compiles to `React.createElement()` calls.

```jsx
// You write:
const element = <h1 className="title">Hello</h1>;

// Babel compiles to:
const element = React.createElement('h1', { className: 'title' }, 'Hello');
```

**Mental model:**  
"JSX is JavaScript with XML-like syntax that creates object descriptions of UI."

---

### 2. **JSX produces React elements (plain objects)**

A React element is just an object describing what you want to see:

```js
{
  type: 'h1',
  props: {
    className: 'title',
    children: 'Hello'
  }
}
```

These objects are cheap to create and compare.

---

### 3. **Virtual DOM = diffing algorithm for efficiency**

React maintains a lightweight copy of the DOM in memory.

**Why?**  
DOM operations are slow. React:
1. Creates new Virtual DOM tree on state change
2. Diffs it with previous Virtual DOM
3. Calculates minimum DOM updates needed
4. Batches updates to real DOM

**Mental model:**  
"React figures out the smallest changeset needed, like Git for UI."

---

### 4. **JSX expressions vs statements**

You can embed **expressions** in JSX, not statements:

```jsx
// ✅ Expressions (evaluate to a value)
<div>{user.name}</div>
<div>{2 + 2}</div>
<div>{user ? 'Logged in' : 'Guest'}</div>

// ❌ Statements (don't evaluate to a value)
<div>{if (user) return user.name}</div>  // Syntax error!
<div>{for (let i = 0; i < 10; i++)}</div>  // Syntax error!
```

Use ternary, logical operators, or extract to functions.

---

### 5. **className, htmlFor, and other differences**

JSX uses camelCase for attributes because it's JavaScript:

| HTML | JSX |
|------|-----|
| `class` | `className` |
| `for` | `htmlFor` |
| `tabindex` | `tabIndex` |
| `onclick` | `onClick` |

This is because `class` and `for` are reserved words in JavaScript.

---

## Examples

### Example 1 — Basic JSX

```jsx
function Greeting({ name, age }) {
  return (
    <div className="greeting">
      <h1>Hello, {name}!</h1>
      <p>You are {age} years old.</p>
    </div>
  );
}
```

**Compiles to:**
```js
function Greeting({ name, age }) {
  return React.createElement('div', { className: 'greeting' },
    React.createElement('h1', null, 'Hello, ', name, '!'),
    React.createElement('p', null, 'You are ', age, ' years old.')
  );
}
```

---

### Example 2 — Conditional rendering

```jsx
function UserStatus({ isLoggedIn, userName }) {
  return (
    <div>
      {isLoggedIn ? (
        <p>Welcome back, {userName}!</p>
      ) : (
        <p>Please log in.</p>
      )}
    </div>
  );
}
```

---

### Example 3 — Inline styles (objects, not strings)

```jsx
function StyledBox() {
  const boxStyle = {
    backgroundColor: 'blue',  // camelCase!
    fontSize: '16px',
    padding: 20  // numbers become 'px'
  };

  return <div style={boxStyle}>Styled!</div>;
}
```

**Key:** Inline styles are JavaScript objects, not CSS strings.

---

### Example 4 — Fragments (avoiding extra divs)

```jsx
// ❌ Extra wrapper div
function List() {
  return (
    <div>
      <li>Item 1</li>
      <li>Item 2</li>
    </div>
  );
}

// ✅ Fragment (no extra DOM node)
function List() {
  return (
    <>
      <li>Item 1</li>
      <li>Item 2</li>
    </>
  );
}

// Also valid:
function List() {
  return (
    <React.Fragment>
      <li>Item 1</li>
      <li>Item 2</li>
    </React.Fragment>
  );
}
```

---

## Common Pitfalls

### 1. **Forgetting to wrap multiple elements**

```jsx
// ❌ Syntax error
function Bad() {
  return
    <h1>Title</h1>
    <p>Content</p>
}

// ✅ Wrapped in fragment
function Good() {
  return (
    <>
      <h1>Title</h1>
      <p>Content</p>
    </>
  );
}
```

---

### 2. **Using `class` instead of `className`**

```jsx
// ❌ Won't work as expected
<div class="container">Content</div>

// ✅ Correct
<div className="container">Content</div>
```

---

### 3. **Self-closing tags must have `/`**

```jsx
// ❌ HTML allows this, JSX doesn't
<input type="text">

// ✅ Must self-close
<input type="text" />
<img src="pic.jpg" />
```

---

### 4. **JavaScript expressions need curly braces**

```jsx
// ❌ Treated as string literal
<div>2 + 2</div>  // Renders: "2 + 2"

// ✅ Evaluated as expression
<div>{2 + 2}</div>  // Renders: "4"
```

---

### 5. **Can't return multiple elements without wrapper**

```jsx
// ❌ Syntax error
return <div>A</div><div>B</div>;

// ✅ Use fragment
return <><div>A</div><div>B</div></>;
```

---

## FAQ

**Q: Do I need React imported for JSX to work?**  
A: With React 17+, no! The new JSX transform auto-imports.  
Before React 17, `import React from 'react'` was required.

**Q: Can I use comments in JSX?**  
A: Yes, use `{/* comment */}` inside JSX.

```jsx
<div>
  {/* This is a comment */}
  <p>Content</p>
</div>
```

**Q: What's the difference between element and component?**  
A: Element = object describing UI (`<div />`).  
Component = function/class that returns elements.

**Q: Why Virtual DOM instead of direct DOM manipulation?**  
A: Predictability, batching, and cross-platform (React Native, SSR).

**Q: Is JSX required for React?**  
A: No, but writing `React.createElement()` manually is verbose.

---

## Quick Self-Check

1. What does this JSX compile to?
   ```jsx
   <button onClick={handleClick}>Click me</button>
   ```

2. Why does React use `className` instead of `class`?

3. What's wrong with this code?
   ```jsx
   function Bad() {
     return
       <div>Hello</div>
   }
   ```

4. How do you render nothing in React?
   ```jsx
   return null;  // or false, undefined
   ```

5. What's the Virtual DOM and why does React use it?

---

## Further Practice

1. **Experiment:** Convert JSX to `React.createElement()` manually
2. **Explore:** Use [Babel REPL](https://babeljs.io/repl) to see JSX transformations
3. **Build:** Create a Card component with conditional rendering
4. **Debug:** Intentionally break JSX syntax and understand errors
5. **Read:** [React Without JSX](https://react.dev/reference/react/createElement)

---

## Key Takeaways

- JSX is syntactic sugar for `React.createElement()` calls
- JSX produces plain JavaScript objects (React elements)
- Virtual DOM enables efficient diffing and batching
- Use expressions (not statements) inside `{}`
- `className`, `htmlFor`, camelCase for attributes
- Fragments avoid extra DOM nodes
