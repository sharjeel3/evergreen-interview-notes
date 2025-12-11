# React Interview Checklist

**Quick reference for interviews** — scan this before technical discussions.

---

## 🎯 Core Concepts (Must Know Cold)

### JSX & Rendering
- [ ] JSX compiles to `React.createElement()`
- [ ] JSX produces plain objects (React elements)
- [ ] Virtual DOM enables efficient diffing
- [ ] Use `className`, `htmlFor` (not `class`, `for`)
- [ ] Expressions in `{}`, not statements
- [ ] Fragments avoid extra DOM nodes (`<>...</>`)

### Components & Props
- [ ] Components are functions that return JSX
- [ ] Props flow one-way (parent → child) and are **read-only**
- [ ] Never mutate props
- [ ] Use `children` prop for composition
- [ ] Function components are standard (classes are legacy)

### State & Lifecycle
- [ ] State is mutable, internal data that triggers re-renders
- [ ] Use `useState`: `const [value, setValue] = useState(initial)`
- [ ] Never mutate state directly—use setter function
- [ ] State updates are **asynchronous and batched**
- [ ] Use functional updates when new state depends on old: `setState(prev => ...)`

### Hooks
- [ ] **Rules of Hooks:** Only at top level, only in React functions
- [ ] `useState` for state, `useEffect` for side effects
- [ ] `useEffect` runs after render (async)
- [ ] Dependency array controls when effect runs: `[]`, `[a]`, or none
- [ ] Return cleanup function from effects (subscriptions, timers, listeners)
- [ ] Missing dependencies cause **stale closures**

### Event Handling
- [ ] React uses SyntheticEvent (cross-browser wrapper)
- [ ] Event names are camelCase: `onClick`, `onChange`
- [ ] Pass function reference, not call: `onClick={fn}` not `onClick={fn()}`
- [ ] Use `e.preventDefault()` and `e.stopPropagation()`
- [ ] React handles event delegation automatically

---

## 🚀 Performance (Common Interview Topic)

### When React Re-renders
- [ ] Component's state changes
- [ ] Props change
- [ ] Parent re-renders (by default, children re-render)
- [ ] Context value changes

### Optimization Tools
- [ ] `React.memo` — Skip re-render if props unchanged (shallow comparison)
- [ ] `useMemo` — Cache expensive calculations: `useMemo(() => calc(a), [a])`
- [ ] `useCallback` — Cache function references: `useCallback(() => fn(), [deps])`
- [ ] `React.lazy` + `Suspense` — Code splitting for large components
- [ ] Virtualization — Render only visible items (react-window)

### Optimization Rules
- [ ] **Measure first** (React DevTools Profiler)
- [ ] Avoid inline objects/arrays as props to memoized components
- [ ] Use stable keys for lists (not array index if reordering)
- [ ] Don't over-optimize—premature optimization adds complexity

---

## 🔥 Advanced Concepts

### useEffect Deep Dive
- [ ] **Runs after render** (doesn't block painting)
- [ ] Empty deps `[]` = run once on mount
- [ ] Cleanup runs before next effect or unmount
- [ ] Can't use async function directly—define inside or use IIFE
- [ ] Race conditions: use cleanup flag or AbortController

### Custom Hooks
- [ ] Extract reusable stateful logic
- [ ] Must start with `use` prefix
- [ ] Follow Rules of Hooks
- [ ] Can call other hooks inside
- [ ] Example: `useFetch`, `useLocalStorage`, `useDebounce`

### Context API
- [ ] Share global state without prop drilling
- [ ] `createContext` → `Context.Provider` → `useContext`
- [ ] **All consumers re-render when context value changes**
- [ ] Optimize: split contexts, memoize value
- [ ] Use for theme, auth, locale (not frequent updates)

### Refs
- [ ] `useRef` persists value without re-render
- [ ] Access DOM: `ref.current`
- [ ] Store mutable values (timers, subscriptions)
- [ ] `forwardRef` passes ref to child component

### Error Boundaries
- [ ] Catch JavaScript errors in component tree
- [ ] Only class components (no hook yet)
- [ ] `componentDidCatch` and `getDerivedStateFromError`
- [ ] Display fallback UI on error

---

## 💬 Common Interview Questions

### Conceptual

**Q: What is React?**  
A: JavaScript library for building UIs. Component-based, declarative, uses Virtual DOM for efficient updates.

**Q: What's the Virtual DOM?**  
A: Lightweight copy of real DOM. React diffs Virtual DOM, calculates minimal changes, batches updates to real DOM.

**Q: Explain one-way data flow.**  
A: Data flows parent → child via props. Child can't modify props. Use callbacks to communicate upward.

**Q: Props vs State?**  
- Props: Passed from parent, read-only, external data
- State: Managed within component, mutable, internal data

**Q: Controlled vs Uncontrolled components?**  
- Controlled: React controls form value via state (`value={state}`)
- Uncontrolled: DOM controls value, access via ref

**Q: Class vs Function components?**  
- Class: Legacy, use `this.state` and lifecycle methods
- Function: Modern, use hooks for state and effects

---

### Hooks Questions

**Q: Why hooks?**  
A: Reuse stateful logic without HOCs/render props. Simpler than class lifecycle. Better code organization.

**Q: Rules of Hooks?**  
1. Only call at top level (not in loops/conditions)
2. Only call from React functions (components or custom hooks)

**Q: When does useEffect run?**  
- No deps: After every render
- Empty `[]`: Once after initial render
- `[a, b]`: After renders where `a` or `b` changed

**Q: Why cleanup function?**  
A: Prevent memory leaks from subscriptions, timers, event listeners. Runs before next effect or unmount.

**Q: What's a stale closure?**  
A: Effect captures old variable value when dependency is missing from array.

**Q: useMemo vs useCallback?**  
- `useMemo`: Returns memoized value
- `useCallback`: Returns memoized function
- `useCallback(fn, deps)` = `useMemo(() => fn, deps)`

---

### Performance Questions

**Q: How to optimize React app?**  
1. Use React DevTools Profiler to identify issues
2. `React.memo` for components with same props
3. `useMemo`/`useCallback` for expensive calculations/callbacks
4. Code splitting with `React.lazy`
5. Virtualize long lists
6. Avoid inline objects/arrays as props

**Q: When does React re-render?**  
- State/props/context change
- Parent re-renders (unless memoized)

**Q: What does React.memo do?**  
A: Shallow compares props. If unchanged, skips re-render.

**Q: Why is key prop important?**  
A: Helps React identify which items changed/added/removed. Enables efficient reconciliation.

---

### Advanced Questions

**Q: How does reconciliation work?**  
A: React diffs new Virtual DOM with previous, calculates minimal DOM updates. Uses keys and component types to optimize.

**Q: What's React Fiber?**  
A: React's reconciliation algorithm (React 16+). Enables incremental rendering, pausing work, prioritizing updates.

**Q: What's concurrent rendering?**  
A: React can interrupt rendering to handle high-priority updates. Keeps UI responsive.

**Q: Explain Context API.**  
A: Share data across component tree without prop drilling. Creates provider-consumer pattern. All consumers re-render when value changes.

**Q: Server-Side Rendering (SSR)?**  
A: Render React on server, send HTML to client. Benefits: SEO, faster first paint. Frameworks: Next.js, Remix.

**Q: What's hydration?**  
A: Attaching React event handlers to server-rendered HTML. Makes static HTML interactive.

---

## 🐛 Common Bugs & How to Fix

| Problem | Cause | Fix |
|---------|-------|-----|
| Infinite loop | State update without deps | Add dependency array |
| Stale closure | Missing deps in effect | Include all deps or use functional update |
| "Can't perform state update on unmounted component" | Async operation after unmount | Cleanup with ignore flag or AbortController |
| Keys warning | Missing/non-unique keys | Use stable, unique key (ID, not index) |
| Component not re-rendering | Mutating state | Use setter with new object/array |
| Memory leak | No cleanup in effect | Return cleanup function |
| Props not updating child | Memoized with unstable deps | Stabilize deps or remove memo |

---

## 🏗️ Patterns & Best Practices

### Component Patterns
- **Composition:** Build complex UIs from simple components
- **Container/Presentational:** Separate logic from presentation
- **Compound Components:** Components that work together (e.g., `<Select><Option /></Select>`)
- **Render Props:** `<Component render={data => <Child data={data} />} />`
- **HOC:** Function that takes component, returns enhanced component (legacy, use hooks)

### State Management
- **Lift state up:** Move state to common ancestor
- **Local first:** Keep state as local as possible
- **Context:** Global state (theme, auth, locale)
- **Libraries:** Redux, Zustand, Jotai for complex state

### Code Organization
- One component per file
- Co-locate related files (component + styles + test)
- Extract reusable logic to custom hooks
- Use TypeScript for type safety

---

## ⚡ Quick Wins for Interviews

1. **Know why hooks exist** (not just how to use them)
2. **Explain Virtual DOM clearly** (why it's efficient)
3. **Demonstrate understanding of dependencies** (useEffect, useMemo, useCallback)
4. **Show you measure before optimizing** (profiling mindset)
5. **Explain one-way data flow** (props down, callbacks up)
6. **Recognize when NOT to optimize** (premature optimization)
7. **Use functional updates** (avoids stale closures)
8. **Clean up effects** (prevent memory leaks)

---

## 🔗 Resources for Deep Dive

- [React Docs (react.dev)](https://react.dev)
- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)
- [Kent C. Dodds Blog](https://kentcdodds.com/blog)
- [React DevTools Profiler](https://react.dev/learn/react-developer-tools)
- [Overreacted (Dan Abramov)](https://overreacted.io/)

---

## 📝 Night-Before Checklist

- [ ] Review Rules of Hooks
- [ ] Practice explaining Virtual DOM
- [ ] Know when React re-renders
- [ ] Understand useEffect dependencies
- [ ] Explain cleanup functions
- [ ] Know optimization tools (memo, useMemo, useCallback)
- [ ] Articulate props vs state
- [ ] Describe one-way data flow
- [ ] Common patterns (composition, lifting state)
- [ ] Debugging approach (DevTools, console, profiling)

---

**Remember:** Interviews test **understanding**, not memorization.  
Explain *why*, not just *how*.
