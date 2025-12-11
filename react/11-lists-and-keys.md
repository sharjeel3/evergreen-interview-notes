# Lists & Keys

**Level:** 🚦 Intermediate  
**Tags:** `lists`, `keys`, `reconciliation`, `performance`, `interview`

---

## Why this matters

Lists and keys are fundamental to React's reconciliation algorithm—how React efficiently updates the DOM. Interviewers frequently ask about keys because improper key usage is one of the most common bugs in React applications, causing performance issues, state bugs, and incorrect UI updates.

**Why interviewers care:**
- Key questions assess understanding of React's Virtual DOM and reconciliation
- Proper key usage shows you understand how React identifies and tracks elements
- Key bugs are notoriously hard to debug and cause subtle production issues
- Senior developers must explain why `key={index}` is dangerous and when it's acceptable
- Understanding keys demonstrates knowledge of React's rendering optimization strategies

**Real-world implications:**
- **Performance:** Incorrect keys force React to destroy/recreate elements unnecessarily
- **State bugs:** Wrong keys can cause component state to be attached to wrong items
- **Animations:** Transitions and animations break when keys change unexpectedly
- **Forms:** Input values can swap between items with incorrect keys
- **Accessibility:** Screen readers and focus management depend on stable element identity
- **User experience:** Items appearing to "swap" or "glitch" during list updates

**Common scenarios with lists:**
- Rendering arrays of data (products, users, comments, todos)
- Dynamic lists (search results, filtered items, sorted data)
- Paginated or infinite scroll lists
- Drag-and-drop reorderable lists
- Nested lists (comments with replies, tree structures)
- Lists with selectable/editable items

**What you must know:**
- **Keys must be stable:** Same item should have same key across renders
- **Keys must be unique:** Among siblings, not globally
- **Don't use array index:** Breaks when items are added/removed/reordered
- **Use stable IDs:** Database IDs, UUIDs, or stable identifiers from data
- **Keys help React reconcile:** Identifies which items changed/added/removed
- **Keys aren't props:** Don't get passed to components (React internal)
- **Fragment keys:** Fragments can have keys when returning multiple elements

**Interview red flag:** Using array index as key without understanding when it's dangerous, or saying "keys are just to remove warnings" shows fundamental misunderstanding of React's reconciliation and will cause production bugs.

---

## Core Ideas

### 1. **Keys help React identify elements across renders**

```jsx
// Without keys, React doesn't know which element is which
<ul>
  <li>Item 1</li>
  <li>Item 2</li>
  <li>Item 3</li>
</ul>

// With keys, React knows element identity
<ul>
  <li key="item-1">Item 1</li>
  <li key="item-2">Item 2</li>
  <li key="item-3">Item 3</li>
</ul>
```

**Mental model:**  
"Keys are like ID badges—React uses them to track which element is which."

---

### 2. **Keys must be unique among siblings**

```jsx
// ✅ Keys unique among siblings
<ul>
  <li key="1">Apple</li>
  <li key="2">Banana</li>
</ul>
<ul>
  <li key="1">Red</li>    {/* OK: different parent */}
  <li key="2">Blue</li>
</ul>

// ❌ Duplicate keys in same parent
<ul>
  <li key="1">Apple</li>
  <li key="1">Banana</li>  {/* ERROR: duplicate key */}
</ul>
```

**Key:** Uniqueness is per parent, not global.

---

### 3. **Array index as key is dangerous**

```jsx
const items = ['Apple', 'Banana', 'Cherry'];

// ❌ Bad: Using index as key
items.map((item, index) => (
  <Item key={index} name={item} />
));

// Problem: If items reorder, keys don't move with data
// Before: key=0 → Apple, key=1 → Banana, key=2 → Cherry
// After reverse: key=0 → Cherry, key=1 → Banana, key=2 → Apple
// React thinks items stayed in place but data changed!

// ✅ Good: Use stable ID from data
items.map((item) => (
  <Item key={item.id} name={item.name} />
));
```

---

### 4. **Keys determine component identity**

```jsx
function Item({ name }) {
  const [count, setCount] = useState(0);
  return (
    <div>
      {name}: {count}
      <button onClick={() => setCount(c => c + 1)}>+</button>
    </div>
  );
}

// ❌ Index keys: State stays with position
const items = ['A', 'B', 'C'];
items.map((item, i) => <Item key={i} name={item} />);
// If you increment B's count, then remove A:
// B moves to position 0 but gets A's old state!

// ✅ Stable keys: State stays with item
items.map(item => <Item key={item.id} name={item.name} />);
// State follows the item regardless of position
```

---

### 5. **Missing keys cause warnings**

```jsx
// ❌ No key (React warning)
{items.map(item => <li>{item}</li>)}

// ✅ Key provided
{items.map(item => <li key={item.id}>{item.name}</li>)}
```

---

## Examples

### Example 1 — Basic list with stable keys

```jsx
function TodoList({ todos }) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          <input 
            type="checkbox" 
            checked={todo.completed} 
          />
          {todo.text}
        </li>
      ))}
    </ul>
  );
}

// Data with stable IDs
const todos = [
  { id: 'todo-1', text: 'Learn React', completed: false },
  { id: 'todo-2', text: 'Build app', completed: false },
  { id: 'todo-3', text: 'Deploy', completed: false }
];
```

---

### Example 2 — Why index keys break

```jsx
function Demo() {
  const [items, setItems] = useState([
    { id: 1, name: 'Alice' },
    { id: 2, name: 'Bob' },
    { id: 3, name: 'Charlie' }
  ]);
  
  const removeFirst = () => setItems(items.slice(1));
  
  return (
    <div>
      <button onClick={removeFirst}>Remove First</button>
      
      {/* ❌ Using index breaks state */}
      {items.map((item, index) => (
        <ListItemWithState key={index} name={item.name} />
      ))}
      
      {/* ✅ Using ID preserves state */}
      {items.map(item => (
        <ListItemWithState key={item.id} name={item.name} />
      ))}
    </div>
  );
}

function ListItemWithState({ name }) {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      {name}: {count}
      <button onClick={() => setCount(c => c + 1)}>+</button>
    </div>
  );
}

/*
With index keys:
1. Increment Bob's count to 5
2. Remove Alice
3. Bob moves to index 0, but gets Alice's state (0)!
4. Bug: Bob's count reset

With ID keys:
1. Increment Bob's count to 5
2. Remove Alice
3. Bob keeps its state (5)
4. Works correctly!
*/
```

---

### Example 3 — Fragments with keys

```jsx
function Glossary({ items }) {
  return (
    <dl>
      {items.map(item => (
        // ✅ Fragment with key (can't use <> shorthand)
        <React.Fragment key={item.id}>
          <dt>{item.term}</dt>
          <dd>{item.description}</dd>
        </React.Fragment>
      ))}
    </dl>
  );
}
```

---

### Example 4 — Nested lists

```jsx
function CommentThread({ comments }) {
  return (
    <ul>
      {comments.map(comment => (
        <li key={comment.id}>
          <Comment data={comment} />
          
          {/* Nested list: Keys unique within this parent */}
          {comment.replies && (
            <ul>
              {comment.replies.map(reply => (
                <li key={reply.id}>
                  <Comment data={reply} />
                </li>
              ))}
            </ul>
          )}
        </li>
      ))}
    </ul>
  );
}
```

---

### Example 5 — When index keys are acceptable

```jsx
// ✅ OK: Static list that never changes
const DAYS = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun'];

function Calendar() {
  return (
    <div>
      {DAYS.map((day, index) => (
        <div key={index}>{day}</div>  // OK: list never changes
      ))}
    </div>
  );
}

// ✅ OK: Pagination where items don't have IDs
function Pagination({ pages }) {
  return (
    <div>
      {pages.map((page, index) => (
        <button key={index}>{index + 1}</button>  // OK: indices are stable
      ))}
    </div>
  );
}

// ❌ NOT OK: Dynamic list that can change
function SearchResults({ results }) {
  return (
    <ul>
      {results.map((result, index) => (
        <li key={index}>{result.name}</li>  // BAD: list changes
      ))}
    </ul>
  );
}
```

---

## Common Pitfalls

### 1. **Using array index when items can change**

```jsx
// ❌ Index breaks when list changes
function TodoList({ todos }) {
  return todos.map((todo, i) => (
    <TodoItem key={i} todo={todo} />  // Bug!
  ));
}

// ✅ Use stable ID
function TodoList({ todos }) {
  return todos.map(todo => (
    <TodoItem key={todo.id} todo={todo} />
  ));
}
```

---

### 2. **Non-unique keys**

```jsx
// ❌ Duplicate keys
const items = [
  { id: 1, name: 'Apple' },
  { id: 1, name: 'Banana' }  // Duplicate ID!
];

items.map(item => <li key={item.id}>{item.name}</li>);
// React warning: duplicate keys

// ✅ Ensure uniqueness
const items = [
  { id: 1, name: 'Apple' },
  { id: 2, name: 'Banana' }
];
```

---

### 3. **Generating keys on the fly**

```jsx
// ❌ New key every render
items.map(item => (
  <Item key={Math.random()} data={item} />  // Always recreates!
));

// ❌ Not stable across renders
items.map(item => (
  <Item key={`${Date.now()}-${item.name}`} data={item} />
));

// ✅ Use stable property
items.map(item => (
  <Item key={item.id} data={item} />
));
```

---

### 4. **Using composite/concatenated keys incorrectly**

```jsx
// ❌ Unnecessary complexity
items.map(item => (
  <Item key={`item-${item.id}-${item.name}`} data={item} />
));

// ✅ Just use ID (simpler and sufficient)
items.map(item => (
  <Item key={item.id} data={item} />
));

// ✅ Composite key when needed (multiple arrays)
[...groupA, ...groupB].map(item => (
  <Item key={`${item.group}-${item.id}`} data={item} />
));
```

---

### 5. **Forgetting keys on fragments**

```jsx
// ❌ No key on fragment in list
items.map(item => (
  <>
    <dt>{item.term}</dt>
    <dd>{item.description}</dd>
  </>
));

// ✅ Fragment with key
items.map(item => (
  <React.Fragment key={item.id}>
    <dt>{item.term}</dt>
    <dd>{item.description}</dd>
  </React.Fragment>
));
```

---

## FAQ

**Q: Why can't I use array index as key?**  
A: Index changes when items reorder/remove. React thinks items stayed but data changed, causing state bugs.

**Q: When IS index acceptable as key?**  
A: When list is static (never changes) or items don't have stable IDs and order doesn't change.

**Q: Do keys need to be globally unique?**  
A: No, only unique among siblings. Same key can exist in different parents.

**Q: Can I use component props as keys?**  
A: No. Keys are React internal—they're not passed to components.

**Q: What if my data doesn't have IDs?**  
A: Generate stable IDs when data arrives (not during render). Libraries: `uuid`, `nanoid`.

**Q: Should I use key to force re-render?**  
A: Sometimes. Changing key recreates component (resets state). Useful for form resets.

**Q: What about keys in conditionally rendered lists?**  
A: Still need keys. Keys must be stable across renders when item is present.

---

## Quick Self-Check

1. Why does React need keys?
   (To identify which items changed/added/removed during reconciliation)

2. What's wrong with `key={index}`?
   ```jsx
   items.map((item, i) => <Item key={i} />)
   ```
   (Breaks when items reorder/remove—state attaches to wrong items)

3. Do keys need to be globally unique?
   (No, only among siblings)

4. What happens if you use duplicate keys?
   (React warning, unpredictable behavior)

5. When is `key={index}` acceptable?
   (Static lists that never change)

---

## Further Practice

1. **Build:** Todo list with add/remove/reorder (test with index vs ID keys)
2. **Debug:** Create bug with index keys, then fix with stable IDs
3. **Experiment:** Stateful list items—observe state bugs with wrong keys
4. **Build:** Sortable table with column sorting (must use stable keys)
5. **Build:** Search/filter list (watch for key warnings)
6. **Read:** React reconciliation algorithm documentation

---

## Key Takeaways

- Keys help React identify elements across renders (reconciliation)
- Keys must be stable (same item = same key) and unique among siblings
- Don't use array index when items can change order or be removed
- Index keys cause state to attach to wrong items after list changes
- Use stable IDs from data (database ID, UUID) as keys
- Keys are React internal—not passed as props to components
- Missing keys cause warnings and performance issues
- Fragments in lists need explicit `<React.Fragment key={...}>`
- Changing key forces component recreation (resets state)
- Index keys are OK for static lists that never change
