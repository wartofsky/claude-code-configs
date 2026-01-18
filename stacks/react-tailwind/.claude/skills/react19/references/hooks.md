# React 19 Hooks Reference

## New Hooks in React 19

### `use(resource)`

Reads the value of a resource (Promise or Context) during render.

```tsx
import { use } from 'react'

// With Promises
function MessageComponent({ messagePromise }) {
  const message = use(messagePromise)
  return <p>{message}</p>
}

// With Context (can be called conditionally!)
function ThemePanel({ showTheme }) {
  if (showTheme) {
    const theme = use(ThemeContext)
    return <div className={theme}>Themed content</div>
  }
  return <div>Default content</div>
}
```

**Key differences from useContext:**
- Can be called inside conditionals and loops
- Works with Promises (suspends until resolved)
- Replaces many use cases for `useContext`

---

### `useActionState(action, initialState, permalink?)`

Manages state based on form action results.

```tsx
import { useActionState } from 'react'

async function increment(previousState, formData) {
  return previousState + 1
}

function Counter() {
  const [state, formAction, isPending] = useActionState(increment, 0)
  
  return (
    <form action={formAction}>
      <p>Count: {state}</p>
      <button disabled={isPending}>
        {isPending ? 'Incrementing...' : 'Increment'}
      </button>
    </form>
  )
}
```

**Parameters:**
- `action`: Function called when form submits
- `initialState`: Initial state value
- `permalink`: Optional URL for progressive enhancement

**Returns:** `[state, formAction, isPending]`

---

### `useFormStatus()`

Returns status information about the parent form.

```tsx
import { useFormStatus } from 'react-dom'

function SubmitButton() {
  const { pending, data, method, action } = useFormStatus()
  
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Submitting...' : 'Submit'}
    </button>
  )
}

// Usage - must be INSIDE a form
function MyForm() {
  return (
    <form action={serverAction}>
      <input name="email" />
      <SubmitButton /> {/* Gets status from parent form */}
    </form>
  )
}
```

**Important:** Must be used in a component that's a child of `<form>`

**Returns:**
- `pending`: boolean - form is submitting
- `data`: FormData | null - submitted data
- `method`: string - HTTP method
- `action`: function | string - form action

---

### `useOptimistic(state, updateFn)`

Shows optimistic state while async action is in progress.

```tsx
import { useOptimistic } from 'react'

function LikeButton({ likes, onLike }) {
  const [optimisticLikes, addOptimisticLike] = useOptimistic(
    likes,
    (currentLikes, increment) => currentLikes + increment
  )
  
  async function handleLike() {
    addOptimisticLike(1)  // Immediately show +1
    await onLike()        // Then persist to server
  }
  
  return (
    <button onClick={handleLike}>
      ❤️ {optimisticLikes}
    </button>
  )
}
```

**Parameters:**
- `state`: Current actual state
- `updateFn`: Pure function that returns optimistic state

**Returns:** `[optimisticState, addOptimistic]`

---

## Updated Hooks

### `useTransition()` (Enhanced)

Now supports async functions and tracks pending state better.

```tsx
import { useTransition } from 'react'

function SearchResults() {
  const [isPending, startTransition] = useTransition()
  const [results, setResults] = useState([])
  
  async function handleSearch(query) {
    startTransition(async () => {
      const data = await searchAPI(query)
      setResults(data)
    })
  }
  
  return (
    <>
      <input onChange={e => handleSearch(e.target.value)} />
      {isPending && <Spinner />}
      <ResultsList results={results} />
    </>
  )
}
```

---

### `useDeferredValue(value, initialValue?)`

Now accepts an optional initial value for first render.

```tsx
import { useDeferredValue } from 'react'

function SearchResults({ query }) {
  // On first render, uses '' instead of query
  // Prevents Suspense from showing on initial load
  const deferredQuery = useDeferredValue(query, '')
  
  return (
    <Suspense fallback={<Skeleton />}>
      <Results query={deferredQuery} />
    </Suspense>
  )
}
```

---

## Classic Hooks (Still Valid)

### State Hooks
- `useState(initialState)` - Local component state
- `useReducer(reducer, initialArg, init?)` - Complex state logic

### Effect Hooks
- `useEffect(setup, deps?)` - Side effects after render
- `useLayoutEffect(setup, deps?)` - Before paint (use sparingly)
- `useInsertionEffect(setup, deps?)` - For CSS-in-JS libraries

### Context
- `useContext(Context)` - Read context value (consider `use()` instead)

### Refs
- `useRef(initialValue)` - Mutable ref that persists across renders
- `useImperativeHandle(ref, createHandle, deps?)` - Customize ref handle

### Performance
- `useMemo(calculateValue, deps)` - Memoize expensive calculations
- `useCallback(fn, deps)` - Memoize callback functions
- `memo(Component)` - Memoize component (not a hook)

### Other
- `useId()` - Generate unique IDs for accessibility
- `useSyncExternalStore(subscribe, getSnapshot)` - Subscribe to external store
- `useDebugValue(value)` - Display value in React DevTools
