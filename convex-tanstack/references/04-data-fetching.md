# Part 4: Data Fetching

## 4.1 Query Patterns

### The Right Import

```typescript
// CORRECT - Use convex-helpers for optimized caching
import { useQuery } from 'convex-helpers/react/cache'

// AVOID - Less optimized
import { useQuery } from 'convex/react'
```

### Basic Query

```typescript
import { useQuery } from 'convex-helpers/react/cache'
import { api } from '~/convex/_generated/api'

function UserList() {
  const users = useQuery(api.users.list)

  // State 1: Loading (undefined)
  if (users === undefined) {
    return <Skeleton className="h-20 rounded" />
  }

  // State 2: Not found or empty (null or [])
  if (!users || users.length === 0) {
    return <div>No users found</div>
  }

  // State 3: Success (data)
  return (
    <ul>
      {users.map((user) => (
        <li key={user._id}>{user.name}</li>
      ))}
    </ul>
  )
}
```

### The "skip" Pattern

Use `"skip"` for conditional queries - **never call hooks conditionally**:

```typescript
import { useQuery } from 'convex-helpers/react/cache'
import { api } from '~/convex/_generated/api'
import type { Id } from '~/convex/_generated/dataModel'

interface UserDetailProps {
  userId?: Id<"users">
}

function UserDetail({ userId }: UserDetailProps) {
  // Skip query if userId is not provided
  const user = useQuery(
    api.users.getById,
    userId ? { userId } : "skip"
  )

  if (user === undefined) return <Skeleton />
  if (user === null) return <div>User not found</div>

  return <div>{user.name}</div>
}
```

### Chained Conditional Queries

```typescript
function Dashboard() {
  // Level 1: Get current user
  const currentUser = useQuery(api.auth.getCurrentUser)

  // Level 2: Get user's organization (depends on user)
  const organization = useQuery(
    api.organizations.getById,
    currentUser?.organizationId ? { id: currentUser.organizationId } : "skip"
  )

  // Level 3: Get organization's projects (depends on org)
  const projects = useQuery(
    api.projects.listByOrg,
    organization?._id ? { organizationId: organization._id } : "skip"
  )

  // Check loading states in order
  if (currentUser === undefined) return <Skeleton />
  if (organization === undefined) return <Skeleton />
  if (projects === undefined) return <Skeleton />

  return <ProjectList projects={projects} />
}
```

### useSuspenseQuery for SSR

Use with route loaders for server-side rendering:

```typescript
import { useSuspenseQuery } from '@tanstack/react-query'
import { convexQuery } from '@convex-dev/react-query'
import { api } from '~/convex/_generated/api'

// In your route file
export const Route = createFileRoute('/posts')({
  loader: async ({ context }) => {
    // Prefetch on the server
    await context.queryClient.ensureQueryData(
      convexQuery(api.posts.list, {})
    )
  },
  component: PostsPage,
})

function PostsPage() {
  // Data is already available from SSR
  const { data: posts } = useSuspenseQuery(
    convexQuery(api.posts.list, {})
  )

  return (
    <ul>
      {posts.map((post) => (
        <li key={post._id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

### Three-State Query Handling

| State | Value | Meaning |
|-------|-------|---------|
| Loading | `undefined` | Query is still fetching |
| Not Found | `null` | Query returned no data |
| Success | `data` | Query returned data |

```typescript
const data = useQuery(api.getData, { id })

// Always check in this order
if (data === undefined) return <LoadingState />
if (data === null) return <NotFoundState />
return <SuccessState data={data} />
```

### Fallback Values

```typescript
// Numbers: use ?? (nullish coalescing)
const count = data?.count ?? 0

// Booleans: use ??
const isActive = data?.isActive ?? false

// Strings: use || (empty string is falsy)
const name = data?.name || "Unknown"

// Arrays: use || (empty array is truthy, but we want default)
const items = data?.items || []
```

---

## 4.2 Mutation Patterns

### Basic Mutation

```typescript
import { useMutation } from 'convex/react'
import { api } from '~/convex/_generated/api'
import { toast } from 'sonner'

function CreateUserButton() {
  const createUser = useMutation(api.users.create)

  const handleClick = async () => {
    try {
      await createUser({ name: 'New User', email: 'user@example.com' })
      toast.success('User created!')
    } catch (error) {
      toast.error(error instanceof Error ? error.message : 'Failed to create user')
    }
  }

  return <button onClick={handleClick}>Create User</button>
}
```

### Mutation with Loading State

```typescript
import { useMutation } from 'convex/react'
import { api } from '~/convex/_generated/api'
import { toast } from 'sonner'
import { useState, useCallback } from 'react'

function DeleteUserButton({ userId }: { userId: Id<"users"> }) {
  const [isDeleting, setIsDeleting] = useState(false)
  const deleteUser = useMutation(api.users.remove)

  const handleDelete = useCallback(async () => {
    setIsDeleting(true)
    try {
      await deleteUser({ userId })
      toast.success('User deleted')
    } catch (error) {
      toast.error(error instanceof Error ? error.message : 'Failed to delete')
    } finally {
      setIsDeleting(false)
    }
  }, [deleteUser, userId])

  return (
    <button onClick={handleDelete} disabled={isDeleting}>
      {isDeleting ? 'Deleting...' : 'Delete'}
    </button>
  )
}
```

### Inline vs Separate Handlers

**Inline** - For simple, single-use operations:

```typescript
const toggleComplete = useMutation(api.todos.toggleComplete)

<button onClick={() => toggleComplete({ todoId: todo._id })}>
  Toggle
</button>
```

**Separate function** - For complex operations or when passed to children:

```typescript
const updateTodo = useMutation(api.todos.update)
const [isUpdating, setIsUpdating] = useState(false)

const handleUpdate = useCallback(async (updates: TodoUpdates) => {
  setIsUpdating(true)
  try {
    await updateTodo({ todoId: todo._id, ...updates })
    toast.success('Updated!')
  } catch (error) {
    toast.error('Failed to update')
  } finally {
    setIsUpdating(false)
  }
}, [updateTodo, todo._id])

return <EditForm onSubmit={handleUpdate} isLoading={isUpdating} />
```

### useConvexMutation with React Query

```typescript
import { useMutation } from '@tanstack/react-query'
import { useConvexMutation } from '@convex-dev/react-query'
import { api } from '~/convex/_generated/api'

function CreatePost() {
  const { mutate, isPending, error } = useMutation({
    mutationFn: useConvexMutation(api.posts.create),
    onSuccess: () => {
      toast.success('Post created!')
    },
    onError: (error) => {
      toast.error(error.message)
    },
  })

  return (
    <form onSubmit={(e) => {
      e.preventDefault()
      const formData = new FormData(e.currentTarget)
      mutate({
        title: formData.get('title') as string,
        content: formData.get('content') as string,
      })
    }}>
      <input name="title" required />
      <textarea name="content" required />
      <button type="submit" disabled={isPending}>
        {isPending ? 'Creating...' : 'Create Post'}
      </button>
      {error && <p className="text-red-500">{error.message}</p>}
    </form>
  )
}
```

---

## 4.3 Loaders & Prefetching

### Route Loader Pattern

```typescript
import { createFileRoute } from '@tanstack/react-router'
import { convexQuery } from '@convex-dev/react-query'
import { useSuspenseQuery } from '@tanstack/react-query'
import { api } from '~/convex/_generated/api'

export const Route = createFileRoute('/posts/$postId')({
  // Runs on hover (preload) and navigation
  loader: async ({ params, context }) => {
    await context.queryClient.ensureQueryData(
      convexQuery(api.posts.getById, { postId: params.postId })
    )
  },
  component: PostPage,
})

function PostPage() {
  const { postId } = Route.useParams()

  // Data is guaranteed to be available (prefetched)
  const { data: post } = useSuspenseQuery(
    convexQuery(api.posts.getById, { postId })
  )

  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  )
}
```

### Multiple Queries in Loader

```typescript
export const Route = createFileRoute('/dashboard')({
  loader: async ({ context }) => {
    // Prefetch multiple queries in parallel
    await Promise.all([
      context.queryClient.ensureQueryData(
        convexQuery(api.users.getCurrentUser, {})
      ),
      context.queryClient.ensureQueryData(
        convexQuery(api.stats.getDashboard, {})
      ),
      context.queryClient.ensureQueryData(
        convexQuery(api.notifications.getRecent, {})
      ),
    ])
  },
  component: DashboardPage,
})
```

### Preload Configuration

```typescript
// In router setup
const router = createRouter({
  routeTree,
  defaultPreload: 'intent',      // Preload on link hover
  defaultPreloadDelay: 50,       // Wait 50ms before preloading
})
```

---

## 4.4 Query Configuration

### gcTime (Garbage Collection Time)

Controls how long subscriptions stay active after component unmounts:

```typescript
const { data } = useQuery({
  ...convexQuery(api.messages.list, {}),
  gcTime: 10000,  // Stay subscribed for 10 seconds after unmount
})
```

Default is 5 minutes (300000ms).

### staleTime

Convex data is never stale (it's real-time), so use:

```typescript
const { data } = useQuery({
  ...convexQuery(api.messages.list, {}),
  staleTime: Infinity,  // Data is never considered stale
})
```

### Ignored Options

These options have no effect with Convex queries:
- `retry` - Convex handles retries internally
- `refetchOnWindowFocus` - Data pushes automatically
- `refetchOnMount` - Subscription resumes automatically
- `refetchInterval` - Real-time updates replace polling

---

