# Forms & Controlled Components

**Level:** 🚦 Intermediate  
**Tags:** `forms`, `controlled-components`, `uncontrolled-components`, `validation`, `interview`

---

## Why this matters

Forms are fundamental to any interactive application, and how you handle them in React reveals your understanding of React's data flow, state management, and performance trade-offs. Interviewers frequently ask about forms because they combine multiple React concepts: state, events, re-rendering, and component design patterns.

**Why interviewers care:**
- Forms demonstrate understanding of React's one-way data flow and controlled components pattern
- Form handling separates candidates who understand state synchronization from those who don't
- The controlled vs. uncontrolled debate tests architectural decision-making
- Validation strategies reveal your approach to user experience and error handling
- Form libraries knowledge shows awareness of production-ready solutions vs. reinventing the wheel

**Real-world implications:**
- **Every app has forms:** Login, registration, checkout, settings, search, comments, etc.
- **User experience:** Instant validation feedback improves UX vs. submit-time errors
- **Data integrity:** Proper validation prevents bad data from reaching your backend
- **Performance:** Large forms with many inputs can cause performance issues if not optimized
- **Accessibility:** Forms must work with keyboards, screen readers, and assistive technology
- **Security:** Client-side validation is UX, not security—always validate server-side too

**Common form challenges:**
- Managing form state (individual fields vs. single object)
- Validation timing (onChange, onBlur, onSubmit)
- Handling file uploads
- Dynamic forms (add/remove fields)
- Multi-step forms / wizards
- Dependent fields (cascade selects)
- Preserving form state on navigation
- Form submission with loading/error states

**What you must know:**
- **Controlled components:** React state is the "single source of truth" for input values
- **Uncontrolled components:** DOM is the source of truth, accessed via refs
- When to use each approach
- Common form libraries (React Hook Form, Formik)
- Validation patterns and libraries (Zod, Yup, native HTML5)

**Interview red flags:** Not knowing the difference between controlled and uncontrolled components, or creating a new function in render for onChange handlers (breaks performance with React.memo).

---

## Core Ideas

### 1. **Controlled Components: React owns the state**

A **controlled component** is an input whose value is controlled by React state.

```jsx
function ControlledInput() {
  const [value, setValue] = useState('');

  return (
    <input
      type="text"
      value={value}  // React state controls the value
      onChange={(e) => setValue(e.target.value)}  // Update state on change
    />
  );
}
```

**How it works:**
1. Input's `value` prop comes from React state
2. User types → `onChange` fires → updates state
3. State update → re-render → input shows new value
4. **React state is the single source of truth**

**Benefits:**
- Instant validation (validate on every keystroke)
- Can transform input (e.g., uppercase, format phone number)
- Easy to reset/clear form programmatically
- Can disable submit until valid
- Predictable state (state always reflects UI)

**Drawbacks:**
- Re-renders on every keystroke (can be slow for large forms)
- More boilerplate code
- Overkill for simple forms

**Example: Live validation**

```jsx
function EmailInput() {
  const [email, setEmail] = useState('');
  const [error, setError] = useState('');

  const handleChange = (e) => {
    const value = e.target.value;
    setEmail(value);
    
    // Live validation
    if (value && !value.includes('@')) {
      setError('Email must contain @');
    } else {
      setError('');
    }
  };

  return (
    <div>
      <input 
        type="email" 
        value={email} 
        onChange={handleChange}
        aria-invalid={!!error}
        aria-describedby="email-error"
      />
      {error && <span id="email-error" className="error">{error}</span>}
    </div>
  );
}
```

---

### 2. **Uncontrolled Components: DOM owns the state**

An **uncontrolled component** stores its own state in the DOM. React accesses it via refs when needed.

```jsx
function UncontrolledInput() {
  const inputRef = useRef(null);

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log(inputRef.current.value);  // Read from DOM
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="text" ref={inputRef} defaultValue="initial" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

**Key differences:**
- Use `ref` instead of `value` prop
- Use `defaultValue` instead of `value` (sets initial value, then DOM takes over)
- No `onChange` needed (unless you want to react to changes)
- Read value with `ref.current.value`

**Benefits:**
- Less code, less boilerplate
- No re-renders on every keystroke (better performance)
- Easier integration with non-React libraries
- Useful for file inputs (must be uncontrolled)

**Drawbacks:**
- Can't validate on every keystroke
- Can't transform input in real-time
- Harder to keep UI in sync with state
- Less "React-like"

**When to use uncontrolled:**
- Simple forms that don't need validation
- File input (always uncontrolled: `<input type="file" />`)
- Integrating with non-React code
- Performance-critical forms with many inputs

---

### 3. **Managing Form State: Individual vs. Object**

**Approach 1: Individual state per field**

```jsx
function LoginForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  
  // Simple but verbose for large forms
}
```

**Approach 2: Single state object**

```jsx
function LoginForm() {
  const [formData, setFormData] = useState({
    email: '',
    password: ''
  });

  const handleChange = (e) => {
    setFormData({
      ...formData,
      [e.target.name]: e.target.value  // Computed property name
    });
  };

  return (
    <form>
      <input
        name="email"  // Must match state key
        value={formData.email}
        onChange={handleChange}
      />
      <input
        name="password"
        type="password"
        value={formData.password}
        onChange={handleChange}
      />
    </form>
  );
}
```

**Benefits of object approach:**
- Scales better for large forms
- Easy to submit (already in object format)
- Single `handleChange` function

**Gotcha:** Spreading object creates new reference on every keystroke, which can break memoization.

---

### 4. **Form Validation Patterns**

**Pattern 1: Validate on submit**

```jsx
function SignupForm() {
  const [formData, setFormData] = useState({ email: '', password: '' });
  const [errors, setErrors] = useState({});

  const validate = () => {
    const newErrors = {};
    
    if (!formData.email.includes('@')) {
      newErrors.email = 'Invalid email';
    }
    
    if (formData.password.length < 8) {
      newErrors.password = 'Password must be 8+ characters';
    }
    
    return newErrors;
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    
    const newErrors = validate();
    if (Object.keys(newErrors).length > 0) {
      setErrors(newErrors);
      return;
    }
    
    // Submit form
    console.log('Submitting:', formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="email"
        value={formData.email}
        onChange={(e) => setFormData({...formData, email: e.target.value})}
      />
      {errors.email && <span className="error">{errors.email}</span>}
      
      <input
        name="password"
        type="password"
        value={formData.password}
        onChange={(e) => setFormData({...formData, password: e.target.value})}
      />
      {errors.password && <span className="error">{errors.password}</span>}
      
      <button type="submit">Sign Up</button>
    </form>
  );
}
```

**Pattern 2: Validate on blur (better UX)**

```jsx
function EmailField() {
  const [email, setEmail] = useState('');
  const [error, setError] = useState('');
  const [touched, setTouched] = useState(false);

  const handleBlur = () => {
    setTouched(true);
    if (!email.includes('@')) {
      setError('Invalid email');
    } else {
      setError('');
    }
  };

  return (
    <>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        onBlur={handleBlur}  // Validate when user leaves field
        aria-invalid={touched && !!error}
      />
      {touched && error && <span className="error">{error}</span>}
    </>
  );
}
```

**Pattern 3: Schema validation with Zod**

```jsx
import { z } from 'zod';

const userSchema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(8, 'Password must be 8+ characters'),
  age: z.number().min(18, 'Must be 18+')
});

function SignupForm() {
  const [formData, setFormData] = useState({ email: '', password: '', age: 0 });
  const [errors, setErrors] = useState({});

  const handleSubmit = (e) => {
    e.preventDefault();
    
    const result = userSchema.safeParse(formData);
    
    if (!result.success) {
      // Convert Zod errors to object
      const fieldErrors = {};
      result.error.issues.forEach(issue => {
        fieldErrors[issue.path[0]] = issue.message;
      });
      setErrors(fieldErrors);
      return;
    }
    
    // Form is valid
    console.log('Valid data:', result.data);
  };

  // ... render form
}
```

---

### 5. **Form Libraries: React Hook Form**

For production apps, use a battle-tested library. **React Hook Form** is the most popular (2024-2025).

**Why React Hook Form?**
- Uses uncontrolled components by default (better performance)
- Integrates with validation libraries (Zod, Yup)
- Minimal re-renders
- Small bundle size (~9KB)
- Great TypeScript support

```jsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  email: z.string().email(),
  password: z.string().min(8)
});

function LoginForm() {
  const { 
    register,      // Register inputs
    handleSubmit,  // Handle submit
    formState: { errors, isSubmitting }  // Form state
  } = useForm({
    resolver: zodResolver(schema)  // Integrate Zod
  });

  const onSubmit = async (data) => {
    // data is already validated
    await fetch('/api/login', {
      method: 'POST',
      body: JSON.stringify(data)
    });
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input 
        {...register('email')}  // Registers with React Hook Form
        type="email"
      />
      {errors.email && <span>{errors.email.message}</span>}
      
      <input 
        {...register('password')}
        type="password"
      />
      {errors.password && <span>{errors.password.message}</span>}
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
}
```

**Advanced: Dynamic fields**

```jsx
import { useForm, useFieldArray } from 'react-hook-form';

function DynamicForm() {
  const { register, control, handleSubmit } = useForm({
    defaultValues: {
      users: [{ name: '', email: '' }]
    }
  });

  const { fields, append, remove } = useFieldArray({
    control,
    name: 'users'
  });

  return (
    <form onSubmit={handleSubmit(console.log)}>
      {fields.map((field, index) => (
        <div key={field.id}>
          <input {...register(`users.${index}.name`)} />
          <input {...register(`users.${index}.email`)} />
          <button type="button" onClick={() => remove(index)}>Remove</button>
        </div>
      ))}
      <button type="button" onClick={() => append({ name: '', email: '' })}>
        Add User
      </button>
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

## Common Pitfalls

### ❌ **Pitfall 1: Creating functions in render**

```jsx
// BAD: Creates new function on every render
function Form() {
  const [value, setValue] = useState('');
  
  return (
    <MemoizedInput 
      value={value}
      onChange={(e) => setValue(e.target.value)}  // New function every render!
    />
  );
}
```

**Why it's bad:** Breaks React.memo/PureComponent optimization.

**Fix: useCallback**

```jsx
function Form() {
  const [value, setValue] = useState('');
  
  const handleChange = useCallback((e) => {
    setValue(e.target.value);
  }, []);  // Stable reference
  
  return <MemoizedInput value={value} onChange={handleChange} />;
}
```

---

### ❌ **Pitfall 2: Forgetting preventDefault**

```jsx
// BAD: Page will reload on submit
function Form() {
  const handleSubmit = () => {
    console.log('Submitting...');  // Never runs (page reloads)
  };
  
  return <form onSubmit={handleSubmit}>...</form>;
}
```

**Fix:**

```jsx
const handleSubmit = (e) => {
  e.preventDefault();  // Prevent page reload
  console.log('Submitting...');
};
```

---

### ❌ **Pitfall 3: Not handling async submit errors**

```jsx
// BAD: No error handling
const handleSubmit = async (e) => {
  e.preventDefault();
  await fetch('/api/submit', { method: 'POST', body: JSON.stringify(data) });
  // What if fetch fails?
};
```

**Fix:**

```jsx
const [error, setError] = useState('');
const [isLoading, setIsLoading] = useState(false);

const handleSubmit = async (e) => {
  e.preventDefault();
  setIsLoading(true);
  setError('');
  
  try {
    const response = await fetch('/api/submit', {
      method: 'POST',
      body: JSON.stringify(data)
    });
    
    if (!response.ok) {
      throw new Error('Submit failed');
    }
    
    // Success
  } catch (err) {
    setError(err.message);
  } finally {
    setIsLoading(false);
  }
};
```

---

### ❌ **Pitfall 4: Using `value` on file inputs**

```jsx
// BAD: File inputs must be uncontrolled
<input type="file" value={file} />  // Error!
```

**Fix:**

```jsx
function FileUpload() {
  const [file, setFile] = useState(null);
  
  const handleChange = (e) => {
    setFile(e.target.files[0]);
  };
  
  return <input type="file" onChange={handleChange} />;  // No value prop
}
```

---

### ❌ **Pitfall 5: Not clearing form after submit**

```jsx
// BAD: Form data remains after submit
const handleSubmit = async (e) => {
  e.preventDefault();
  await submitData(formData);
  // Form still shows old data
};
```

**Fix:**

```jsx
const handleSubmit = async (e) => {
  e.preventDefault();
  await submitData(formData);
  setFormData({ email: '', password: '' });  // Clear form
};
```

---

## Quick Self-Check

✅ **You understand forms if you can:**

1. Explain the difference between controlled and uncontrolled components
2. Write a form with validation (email, password, confirm password)
3. Explain when you'd use React Hook Form vs. plain React state
4. Handle form submission with loading and error states
5. Explain why `e.preventDefault()` is needed in `onSubmit`
6. Build a multi-step form / wizard
7. Implement dependent fields (e.g., country → state/province cascade)
8. Handle file uploads
9. Explain the performance trade-offs of controlled components
10. Integrate a validation library (Zod, Yup)

---

## Interview Questions to Practice

**Beginner:**
1. What's the difference between controlled and uncontrolled components?
2. What's the difference between `value` and `defaultValue`?
3. Why do we need `e.preventDefault()` in form submit handlers?

**Intermediate:**
4. How would you implement form validation that runs on blur?
5. How do you handle form submission with async API calls?
6. When would you use React Hook Form instead of plain useState?
7. How do you optimize a form with many inputs to avoid unnecessary re-renders?

**Advanced:**
8. Build a dynamic form where users can add/remove fields
9. Implement a multi-step wizard with validation on each step
10. How would you persist form data when navigating between pages?

---

## Further Practice

**Build these:**
1. **Login form** with email/password validation
2. **Signup form** with password confirmation and strength indicator
3. **Multi-step wizard** (personal info → address → review → submit)
4. **Dynamic form** where users can add multiple addresses
5. **Search form** with debounced autocomplete
6. **File upload** with preview and progress bar
7. **Nested form** (company with multiple employees, each with multiple skills)

**Resources:**
- [React Hook Form docs](https://react-hook-form.com/)
- [Zod validation library](https://zod.dev/)
- [Formik (alternative to RHF)](https://formik.org/)
- [HTML5 form validation](https://developer.mozilla.org/en-US/docs/Learn/Forms/Form_validation)

---

## Summary

**Controlled components:**
- React state is source of truth
- `value` + `onChange`
- Best for: validation, formatting, real-time feedback

**Uncontrolled components:**
- DOM is source of truth
- `ref` + `defaultValue`
- Best for: simple forms, file inputs, performance

**Production forms:**
- Use React Hook Form (uncontrolled by default, better performance)
- Use validation schema (Zod, Yup)
- Handle loading, error, and success states
- Always validate server-side (client validation is UX, not security)

**Performance:**
- Controlled components re-render on every keystroke
- Use `useCallback` for onChange handlers in optimized components
- Consider React Hook Form for large forms

**Remember:** Client-side validation improves UX, but always validate on the server for security.
