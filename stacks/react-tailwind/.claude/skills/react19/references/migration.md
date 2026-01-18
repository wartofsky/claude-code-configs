# Migration Guide: React 18 → React 19

## Breaking Changes

### 1. `useFormState` → `useActionState`

```tsx
// ❌ React 18 (react-dom)
import { useFormState } from 'react-dom'
const [state, formAction] = useFormState(action, initialState)

// ✅ React 19 (react)
import { useActionState } from 'react'
const [state, formAction, isPending] = useActionState(action, initialState)
```

**Changes:**
- Moved from `react-dom` to `react`
- Renamed to `useActionState`
- Now returns `isPending` as third element

### 2. `ref` as a Prop

```tsx
// ❌ React 18 - needed forwardRef
const Input = forwardRef((props, ref) => (
  <input ref={ref} {...props} />
))

// ✅ React 19 - ref is just a prop
function Input({ ref, ...props }) {
  return <input ref={ref} {...props} />
}
```

`forwardRef` is deprecated. Simply accept `ref` as a prop.

### 3. Context as Provider

```tsx
// ❌ React 18
<ThemeContext.Provider value={theme}>
  {children}
</ThemeContext.Provider>

// ✅ React 19
<ThemeContext value={theme}>
  {children}
</ThemeContext>
```

### 4. Cleanup Functions in `ref` Callbacks

```tsx
// ✅ React 19 - return cleanup function
<div ref={(node) => {
  // Setup
  node.addEventListener('click', handler)
  
  // Cleanup (new!)
  return () => {
    node.removeEventListener('click', handler)
  }
}} />
```

### 5. `useDeferredValue` Initial Value

```tsx
// ❌ React 18
const deferredValue = useDeferredValue(value)

// ✅ React 19 - optional initial value
const deferredValue = useDeferredValue(value, initialValue)
```

---

## Deprecated APIs

| Deprecated | Replacement |
|------------|-------------|
| `forwardRef` | Pass `ref` as prop |
| `<Context.Provider>` | `<Context>` directly |
| `useFormState` | `useActionState` |
| `ReactDOM.render` | `createRoot().render()` |
| `ReactDOM.hydrate` | `hydrateRoot()` |
| `unmountComponentAtNode` | `root.unmount()` |
| `renderToString` (sync) | Use streaming APIs |

---

## New Features to Adopt

### Document Metadata

```tsx
// ❌ Before - needed react-helmet or next/head
import { Helmet } from 'react-helmet'
<Helmet>
  <title>My Page</title>
</Helmet>

// ✅ React 19 - native support
function BlogPost({ post }) {
  return (
    <article>
      <title>{post.title}</title>
      <meta name="description" content={post.summary} />
      <link rel="canonical" href={post.url} />
      <h1>{post.title}</h1>
    </article>
  )
}
```

### Stylesheet Precedence

```tsx
// React 19 - stylesheets with precedence
<link 
  rel="stylesheet" 
  href="styles.css" 
  precedence="default"  // Controls load order
/>
```

### Async Scripts

```tsx
// React 19 - automatic deduplication
function Component() {
  return (
    <>
      <script async src="analytics.js" />
      {/* Won't load twice even if rendered multiple times */}
    </>
  )
}
```

### Preloading Resources

```tsx
import { prefetchDNS, preconnect, preload, preinit } from 'react-dom'

function App() {
  // Preload resources
  prefetchDNS('https://api.example.com')
  preconnect('https://cdn.example.com')
  preload('https://cdn.example.com/font.woff2', { as: 'font' })
  preinit('https://cdn.example.com/script.js', { as: 'script' })
  
  return <MyApp />
}
```

---

## Migration Checklist

### Phase 1: Update Dependencies

```bash
npm install react@19 react-dom@19
npm install -D @types/react@19 @types/react-dom@19
```

### Phase 2: Fix Breaking Changes

- [ ] Replace `useFormState` with `useActionState`
- [ ] Remove `forwardRef` wrappers, use `ref` prop
- [ ] Update Context providers to new syntax
- [ ] Add cleanup returns to ref callbacks if needed

### Phase 3: Update Deprecated APIs

- [ ] Replace `ReactDOM.render` with `createRoot`
- [ ] Replace `ReactDOM.hydrate` with `hydrateRoot`
- [ ] Replace `unmountComponentAtNode` with `root.unmount()`

### Phase 4: Adopt New Features

- [ ] Use native `<title>`, `<meta>` in components
- [ ] Replace react-helmet if used
- [ ] Use `use()` hook where appropriate
- [ ] Implement optimistic updates with `useOptimistic`
- [ ] Convert forms to use Server Actions

---

## TypeScript Changes

### Types Package

```bash
npm install -D @types/react@19 @types/react-dom@19
```

### Ref Types

```tsx
// ❌ Before
const ref = useRef<HTMLDivElement | null>(null)

// ✅ React 19 - null handling improved
const ref = useRef<HTMLDivElement>(null)
```

### Action State Types

```tsx
type FormState = {
  success: boolean
  message: string
}

const [state, action, isPending] = useActionState<FormState, FormData>(
  async (prevState, formData) => {
    // ...
    return { success: true, message: 'Done!' }
  },
  { success: false, message: '' }
)
```

---

## Common Migration Issues

### Issue: `forwardRef` type errors

```tsx
// Fix: Remove forwardRef, add ref to props
interface Props {
  ref?: React.Ref<HTMLInputElement>
  // other props
}

function Input({ ref, ...props }: Props) {
  return <input ref={ref} {...props} />
}
```

### Issue: Context Provider errors

```tsx
// Fix: Remove .Provider
// Old: <MyContext.Provider value={...}>
// New: <MyContext value={...}>
```

### Issue: useFormState import errors

```tsx
// Fix: Update import and name
// Old: import { useFormState } from 'react-dom'
// New: import { useActionState } from 'react'
```
