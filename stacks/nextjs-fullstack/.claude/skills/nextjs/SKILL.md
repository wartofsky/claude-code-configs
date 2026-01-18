---
name: nextjs15
description: Next.js 15 patterns with App Router, Server Components, Server Actions, and caching strategies. Use when building Next.js pages, API routes, data fetching, or server-side logic. Triggers on Next.js, App Router, Server Action, RSC, or routing keywords.
---

# Next.js 15 Patterns

## App Router Structure

```
app/
├── layout.tsx           # Root layout (required)
├── page.tsx             # Home page (/)
├── loading.tsx          # Loading UI
├── error.tsx            # Error UI
├── not-found.tsx        # 404 UI
├── (group)/             # Route group (no URL segment)
│   └── page.tsx
├── folder/
│   ├── page.tsx         # /folder
│   └── [id]/
│       └── page.tsx     # /folder/:id
└── api/
    └── route.ts         # API endpoint
```

## Server vs Client Components

| Feature | Server Component | Client Component |
|---------|------------------|------------------|
| Default | ✅ Yes | ❌ Needs 'use client' |
| async/await | ✅ Yes | ❌ No |
| useState/useEffect | ❌ No | ✅ Yes |
| Event handlers | ❌ No | ✅ Yes |
| Browser APIs | ❌ No | ✅ Yes |
| Database direct | ✅ Yes | ❌ No |

## Data Fetching

### Server Component (Recommended)

```tsx
// Direct async in component
async function Page() {
  const data = await db.query()
  return <Component data={data} />
}
```

### With Caching

```tsx
// Time-based revalidation
const data = await fetch(url, { 
  next: { revalidate: 3600 }  // 1 hour
})

// Tag-based revalidation
const data = await fetch(url, {
  next: { tags: ['posts'] }
})

// Revalidate after mutation
import { revalidateTag } from 'next/cache'
revalidateTag('posts')
```

### Parallel Fetching

```tsx
async function Page() {
  // ✅ Parallel - fast
  const [posts, users] = await Promise.all([
    fetchPosts(),
    fetchUsers()
  ])
  
  // ❌ Sequential - slow
  const posts = await fetchPosts()
  const users = await fetchUsers()
}
```

## Server Actions

```tsx
// actions/posts.ts
'use server'

import { revalidatePath } from 'next/cache'
import { redirect } from 'next/navigation'

export async function createPost(formData: FormData) {
  const title = formData.get('title')
  
  await db.post.create({ data: { title } })
  
  revalidatePath('/posts')
  redirect('/posts')
}

// Usage in form
<form action={createPost}>
  <input name="title" />
  <button type="submit">Create</button>
</form>
```

### With useActionState

```tsx
'use client'
import { useActionState } from 'react'

function Form() {
  const [state, action, isPending] = useActionState(createPost, null)
  
  return (
    <form action={action}>
      {state?.error && <p>{state.error}</p>}
      <input name="title" disabled={isPending} />
      <button disabled={isPending}>
        {isPending ? 'Creating...' : 'Create'}
      </button>
    </form>
  )
}
```

## Route Handlers

```tsx
// app/api/posts/route.ts
import { NextRequest, NextResponse } from 'next/server'

export async function GET(request: NextRequest) {
  const { searchParams } = request.nextUrl
  const page = searchParams.get('page')
  
  const data = await fetchData(page)
  
  return NextResponse.json({ data })
}

export async function POST(request: NextRequest) {
  const body = await request.json()
  
  const result = await createData(body)
  
  return NextResponse.json({ data: result }, { status: 201 })
}
```

### Dynamic Route Handler

```tsx
// app/api/posts/[id]/route.ts
interface Context {
  params: Promise<{ id: string }>
}

export async function GET(
  request: NextRequest,
  context: Context
) {
  const { id } = await context.params
  // ...
}
```

## Layouts & Templates

```tsx
// layout.tsx - Persists across navigations
export default function Layout({ children }) {
  return (
    <div>
      <Sidebar />  {/* Doesn't re-render on navigation */}
      {children}
    </div>
  )
}

// template.tsx - Re-mounts on navigation
export default function Template({ children }) {
  return <div>{children}</div>
}
```

## Loading & Error States

```tsx
// loading.tsx - Automatic Suspense
export default function Loading() {
  return <Skeleton />
}

// error.tsx - Error boundary
'use client'
export default function Error({ error, reset }) {
  return (
    <div>
      <p>Error: {error.message}</p>
      <button onClick={reset}>Retry</button>
    </div>
  )
}
```

## Middleware

```tsx
// middleware.ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  // Auth check
  const token = request.cookies.get('token')
  
  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url))
  }
  
  return NextResponse.next()
}

export const config = {
  matcher: ['/dashboard/:path*']
}
```

## Metadata

```tsx
// Static
export const metadata = {
  title: 'Page Title',
  description: 'Description'
}

// Dynamic
export async function generateMetadata({ params }) {
  const post = await getPost(params.id)
  return { title: post.title }
}
```

## Image Optimization

```tsx
import Image from 'next/image'

<Image
  src="/photo.jpg"
  alt="Description"
  width={800}
  height={600}
  priority  // For LCP images
/>
```
