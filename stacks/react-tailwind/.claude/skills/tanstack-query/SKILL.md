---
name: tanstack-query
description: TanStack Query (React Query) patterns for data fetching, caching, and server state management. Use when implementing API calls, caching strategies, optimistic updates, or infinite scroll. Triggers on useQuery, useMutation, React Query, data fetching, cache keywords.
---

# TanStack Query Patterns

## Setup

```tsx
// providers.tsx
'use client'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'
import { useState } from 'react'

export function Providers({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(
    () =>
      new QueryClient({
        defaultOptions: {
          queries: {
            staleTime: 60 * 1000, // 1 minute
            gcTime: 5 * 60 * 1000, // 5 minutes (formerly cacheTime)
            retry: 1,
            refetchOnWindowFocus: false,
          },
        },
      })
  )

  return (
    <QueryClientProvider client={queryClient}>
      {children}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  )
}
```

## Basic Query

```tsx
import { useQuery } from '@tanstack/react-query'

interface User {
  id: string
  name: string
  email: string
}

// API function
async function fetchUser(userId: string): Promise<User> {
  const res = await fetch(`/api/users/${userId}`)
  if (!res.ok) throw new Error('Failed to fetch user')
  return res.json()
}

// Component
function UserProfile({ userId }: { userId: string }) {
  const { data, isLoading, isError, error } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  })

  if (isLoading) return <Skeleton />
  if (isError) return <ErrorMessage error={error} />

  return (
    <div>
      <h1>{data.name}</h1>
      <p>{data.email}</p>
    </div>
  )
}
```

## Query Keys

```tsx
// Simple key
queryKey: ['todos']

// With parameters
queryKey: ['todo', todoId]

// With filters
queryKey: ['todos', { status: 'completed', page: 1 }]

// Hierarchical (for invalidation)
queryKey: ['users', userId, 'posts']
// Invalidate all user data: invalidateQueries({ queryKey: ['users', userId] })
```

## Mutations

```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query'

interface CreateTodoInput {
  title: string
  completed: boolean
}

function TodoForm() {
  const queryClient = useQueryClient()

  const mutation = useMutation({
    mutationFn: async (newTodo: CreateTodoInput) => {
      const res = await fetch('/api/todos', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(newTodo),
      })
      if (!res.ok) throw new Error('Failed to create todo')
      return res.json()
    },
    onSuccess: () => {
      // Invalidate and refetch
      queryClient.invalidateQueries({ queryKey: ['todos'] })
    },
    onError: (error) => {
      toast.error(`Error: ${error.message}`)
    },
  })

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault()
    const formData = new FormData(e.currentTarget)
    mutation.mutate({
      title: formData.get('title') as string,
      completed: false,
    })
  }

  return (
    <form onSubmit={handleSubmit}>
      <input name="title" required />
      <button type="submit" disabled={mutation.isPending}>
        {mutation.isPending ? 'Adding...' : 'Add Todo'}
      </button>
    </form>
  )
}
```

## Optimistic Updates

```tsx
const mutation = useMutation({
  mutationFn: updateTodo,
  onMutate: async (newTodo) => {
    // Cancel outgoing refetches
    await queryClient.cancelQueries({ queryKey: ['todos'] })

    // Snapshot previous value
    const previousTodos = queryClient.getQueryData(['todos'])

    // Optimistically update
    queryClient.setQueryData(['todos'], (old: Todo[]) =>
      old.map((todo) =>
        todo.id === newTodo.id ? { ...todo, ...newTodo } : todo
      )
    )

    // Return context for rollback
    return { previousTodos }
  },
  onError: (err, newTodo, context) => {
    // Rollback on error
    queryClient.setQueryData(['todos'], context?.previousTodos)
  },
  onSettled: () => {
    // Always refetch after error or success
    queryClient.invalidateQueries({ queryKey: ['todos'] })
  },
})
```

## Infinite Queries (Pagination)

```tsx
import { useInfiniteQuery } from '@tanstack/react-query'

interface PostsPage {
  posts: Post[]
  nextCursor: string | null
}

function PostsList() {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
    isLoading,
  } = useInfiniteQuery({
    queryKey: ['posts'],
    queryFn: async ({ pageParam }): Promise<PostsPage> => {
      const res = await fetch(`/api/posts?cursor=${pageParam}`)
      return res.json()
    },
    initialPageParam: '',
    getNextPageParam: (lastPage) => lastPage.nextCursor,
  })

  if (isLoading) return <Skeleton />

  return (
    <div>
      {data?.pages.map((page, i) => (
        <div key={i}>
          {page.posts.map((post) => (
            <PostCard key={post.id} post={post} />
          ))}
        </div>
      ))}

      <button
        onClick={() => fetchNextPage()}
        disabled={!hasNextPage || isFetchingNextPage}
      >
        {isFetchingNextPage
          ? 'Loading more...'
          : hasNextPage
          ? 'Load More'
          : 'Nothing more to load'}
      </button>
    </div>
  )
}
```

## Parallel Queries

```tsx
import { useQueries } from '@tanstack/react-query'

function Dashboard({ userIds }: { userIds: string[] }) {
  const results = useQueries({
    queries: userIds.map((id) => ({
      queryKey: ['user', id],
      queryFn: () => fetchUser(id),
    })),
  })

  const isLoading = results.some((r) => r.isLoading)
  const users = results.map((r) => r.data).filter(Boolean)

  if (isLoading) return <Skeleton />

  return (
    <div>
      {users.map((user) => (
        <UserCard key={user.id} user={user} />
      ))}
    </div>
  )
}
```

## Dependent Queries

```tsx
function UserPosts({ userId }: { userId: string }) {
  // First query
  const { data: user } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  })

  // Dependent query - only runs when user is available
  const { data: posts } = useQuery({
    queryKey: ['posts', user?.id],
    queryFn: () => fetchUserPosts(user!.id),
    enabled: !!user, // Only run when user exists
  })

  return <PostList posts={posts ?? []} />
}
```

## Prefetching

```tsx
// Prefetch on hover
function PostLink({ postId }: { postId: string }) {
  const queryClient = useQueryClient()

  const prefetch = () => {
    queryClient.prefetchQuery({
      queryKey: ['post', postId],
      queryFn: () => fetchPost(postId),
      staleTime: 5 * 60 * 1000, // Only prefetch if older than 5 min
    })
  }

  return (
    <Link href={`/posts/${postId}`} onMouseEnter={prefetch}>
      View Post
    </Link>
  )
}

// Prefetch in loader (Next.js)
export async function generateMetadata({ params }: Props) {
  const queryClient = new QueryClient()

  await queryClient.prefetchQuery({
    queryKey: ['post', params.id],
    queryFn: () => fetchPost(params.id),
  })

  // ...
}
```

## Query Invalidation

```tsx
const queryClient = useQueryClient()

// Invalidate all queries
queryClient.invalidateQueries()

// Invalidate specific query
queryClient.invalidateQueries({ queryKey: ['todos'] })

// Invalidate with parameters
queryClient.invalidateQueries({ queryKey: ['todo', todoId] })

// Invalidate hierarchically
queryClient.invalidateQueries({ queryKey: ['users', userId] })
// Invalidates ['users', userId], ['users', userId, 'posts'], etc.

// Refetch (bypass stale check)
queryClient.refetchQueries({ queryKey: ['todos'] })

// Remove from cache entirely
queryClient.removeQueries({ queryKey: ['todos'] })
```

## Error Handling

```tsx
// Global error handler
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: (failureCount, error) => {
        // Don't retry on 404
        if (error instanceof Error && error.message.includes('404')) {
          return false
        }
        return failureCount < 3
      },
    },
    mutations: {
      onError: (error) => {
        toast.error(`Error: ${error.message}`)
      },
    },
  },
})

// Per-query error handling
const { data, error, isError } = useQuery({
  queryKey: ['user', userId],
  queryFn: () => fetchUser(userId),
  throwOnError: false, // Handle error in component
})

// With Error Boundary
<QueryErrorResetBoundary>
  {({ reset }) => (
    <ErrorBoundary onReset={reset} fallbackRender={ErrorFallback}>
      <UserProfile />
    </ErrorBoundary>
  )}
</QueryErrorResetBoundary>
```

## Suspense Mode

```tsx
import { useSuspenseQuery } from '@tanstack/react-query'
import { Suspense } from 'react'

function UserProfile({ userId }: { userId: string }) {
  // This will suspend until data is ready
  const { data } = useSuspenseQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  })

  // data is guaranteed to be defined
  return <div>{data.name}</div>
}

// Usage
<Suspense fallback={<Skeleton />}>
  <UserProfile userId="123" />
</Suspense>
```

## Best Practices

1. **Consistent query keys** - Use arrays, put entity type first
2. **Colocate queries** - Keep queries near components that use them
3. **Custom hooks** - Abstract queries into reusable hooks
4. **Stale time** - Set appropriate stale times to reduce refetches
5. **Optimistic updates** - For better UX on mutations
6. **Error boundaries** - Use for graceful error handling
7. **Devtools** - Use in development for debugging

## Custom Hook Pattern

```tsx
// hooks/useUser.ts
export function useUser(userId: string) {
  return useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
    staleTime: 5 * 60 * 1000,
  })
}

export function useUpdateUser() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: updateUser,
    onSuccess: (data) => {
      queryClient.setQueryData(['user', data.id], data)
    },
  })
}

// Usage
function Profile({ userId }: { userId: string }) {
  const { data: user, isLoading } = useUser(userId)
  const updateUser = useUpdateUser()

  // ...
}
```
