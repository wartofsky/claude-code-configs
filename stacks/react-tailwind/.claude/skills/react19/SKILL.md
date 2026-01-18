---
name: react19
description: React 19 development patterns and best practices. Use when creating React components, handling forms, managing state, fetching data, or working with Server Components and Server Actions. Triggers on React, component, hook, form, useState, useEffect, Server Component, or RSC keywords.
---

# React 19 Development Skill

## Quick Reference

| Feature | Purpose | Import |
|---------|---------|--------|
| `use()` | Read promises/context in render | `import { use } from 'react'` |
| `useActionState` | Form action state | `import { useActionState } from 'react'` |
| `useFormStatus` | Form submission status | `import { useFormStatus } from 'react-dom'` |
| `useOptimistic` | Optimistic UI updates | `import { useOptimistic } from 'react'` |
| `useTransition` | Non-blocking updates | `import { useTransition } from 'react'` |

## Component Patterns

### Functional Component Template

```tsx
interface Props {
  title: string
  onAction?: () => void
  children?: React.ReactNode
}

export function ComponentName({ title, onAction, children }: Props) {
  // 1. Hooks
  const [state, setState] = useState<string>('')
  
  // 2. Derived state
  const isValid = state.length > 0
  
  // 3. Handlers
  const handleSubmit = () => {
    if (isValid && onAction) onAction()
  }
  
  // 4. Render
  return (
    <div>
      <h2>{title}</h2>
      {children}
    </div>
  )
}
```

### Server vs Client Components

```tsx
// SERVER COMPONENT (default) - runs on server, no state/effects
async function ServerComponent() {
  const data = await db.query('SELECT * FROM posts')
  return <PostList posts={data} />
}

// CLIENT COMPONENT - requires directive, runs in browser
'use client'
import { useState } from 'react'

function ClientComponent() {
  const [count, setCount] = useState(0)
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>
}
```

**When to use 'use client':**
- useState, useEffect, useReducer
- Event handlers (onClick, onChange)
- Browser APIs (localStorage, window)
- Third-party client libraries

## The `use()` Hook

Read promises and context directly in render:

```tsx
import { use, Suspense } from 'react'

// Reading a promise
function Comments({ commentsPromise }: { commentsPromise: Promise<Comment[]> }) {
  const comments = use(commentsPromise)
  return (
    <ul>
      {comments.map(c => <li key={c.id}>{c.text}</li>)}
    </ul>
  )
}

// Usage with Suspense
function Page() {
  return (
    <Suspense fallback={<Skeleton />}>
      <Comments commentsPromise={fetchComments()} />
    </Suspense>
  )
}

// Reading context conditionally (NEW - couldn't do with useContext)
function Panel({ showTheme }: { showTheme: boolean }) {
  if (showTheme) {
    const theme = use(ThemeContext)
    return <div style={{ color: theme.primary }}>Themed</div>
  }
  return <div>No theme</div>
}
```

## Form Handling (React 19 Way)

### Basic Form with Server Action

```tsx
// actions.ts
'use server'

export async function submitForm(formData: FormData) {
  const email = formData.get('email') as string
  // Validate and save...
  return { success: true }
}

// Form.tsx
'use client'
import { useActionState } from 'react'
import { useFormStatus } from 'react-dom'
import { submitForm } from './actions'

function SubmitButton() {
  const { pending } = useFormStatus()
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Saving...' : 'Submit'}
    </button>
  )
}

export function MyForm() {
  const [state, action, isPending] = useActionState(submitForm, null)
  
  return (
    <form action={action}>
      <input name="email" type="email" disabled={isPending} />
      <SubmitButton />
      {state?.success && <p>Saved!</p>}
    </form>
  )
}
```

### Optimistic Updates

```tsx
'use client'
import { useOptimistic } from 'react'

function TodoList({ todos }: { todos: Todo[] }) {
  const [optimisticTodos, addOptimistic] = useOptimistic(
    todos,
    (state, newTodo: Todo) => [...state, newTodo]
  )
  
  async function addTodo(formData: FormData) {
    const title = formData.get('title') as string
    const tempTodo = { id: 'temp', title, completed: false }
    
    // Instantly update UI
    addOptimistic(tempTodo)
    
    // Then persist to server
    await saveTodo(title)
  }
  
  return (
    <form action={addTodo}>
      <input name="title" />
      <ul>
        {optimisticTodos.map(todo => (
          <li key={todo.id}>{todo.title}</li>
        ))}
      </ul>
    </form>
  )
}
```

## Data Fetching Patterns

### Server Component (Recommended)

```tsx
// page.tsx (Server Component)
async function Page({ params }: { params: { id: string } }) {
  const post = await fetch(`/api/posts/${params.id}`).then(r => r.json())
  
  return (
    <article>
      <h1>{post.title}</h1>
      <Suspense fallback={<CommentsSkeleton />}>
        <Comments postId={params.id} />
      </Suspense>
    </article>
  )
}

async function Comments({ postId }: { postId: string }) {
  const comments = await fetch(`/api/posts/${postId}/comments`).then(r => r.json())
  return <CommentList comments={comments} />
}
```

### Client-Side with TanStack Query

```tsx
'use client'
import { useQuery } from '@tanstack/react-query'

function UserProfile({ userId }: { userId: string }) {
  const { data, isLoading, error } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetch(`/api/users/${userId}`).then(r => r.json())
  })
  
  if (isLoading) return <Skeleton />
  if (error) return <ErrorMessage error={error} />
  
  return <ProfileCard user={data} />
}
```

## Error Handling

### Error Boundaries

```tsx
'use client'
import { Component, ErrorInfo, ReactNode } from 'react'

interface Props {
  children: ReactNode
  fallback?: ReactNode
}

interface State {
  hasError: boolean
  error?: Error
}

export class ErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false }
  
  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error }
  }
  
  componentDidCatch(error: Error, info: ErrorInfo) {
    console.error('Error caught:', error, info)
  }
  
  render() {
    if (this.state.hasError) {
      return this.props.fallback || <DefaultErrorUI error={this.state.error} />
    }
    return this.props.children
  }
}
```

### Next.js error.tsx

```tsx
'use client'

export default function Error({
  error,
  reset
}: {
  error: Error
  reset: () => void
}) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={reset}>Try again</button>
    </div>
  )
}
```

## Performance Patterns

```tsx
// Memoize expensive components
const MemoizedComponent = memo(ExpensiveComponent)

// Memoize expensive calculations
const sortedItems = useMemo(
  () => items.sort((a, b) => a.name.localeCompare(b.name)),
  [items]
)

// Stable callback references
const handleClick = useCallback((id: string) => {
  setSelectedId(id)
}, [])

// Non-blocking state updates
const [isPending, startTransition] = useTransition()

function handleSearch(query: string) {
  startTransition(() => {
    setSearchResults(filterResults(query))
  })
}
```

## References

For detailed hook documentation, see [references/hooks.md](references/hooks.md)
For migration guide from React 18, see [references/migration.md](references/migration.md)
