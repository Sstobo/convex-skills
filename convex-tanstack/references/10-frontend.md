# Part 10: Frontend Patterns

## 10.1 Component Patterns

### Standard Component Structure

```typescript
import React, { useState, useCallback } from 'react'
import { useQuery } from 'convex-helpers/react/cache'
import { useMutation } from 'convex/react'
import { api } from '~/convex/_generated/api'
import { Skeleton } from '~/components/ui/skeleton'
import { Button } from '~/components/ui/button'
import { toast } from 'sonner'
import type { Id } from '~/convex/_generated/dataModel'

interface UserCardProps {
  userId: Id<"users">
  onUpdate?: () => void
}

export function UserCard({ userId, onUpdate }: UserCardProps) {
  // 1. Data fetching
  const user = useQuery(api.users.getById, { userId })
  const updateUser = useMutation(api.users.update)

  // 2. Local state
  const [isEditing, setIsEditing] = useState(false)

  // 3. Handlers
  const handleSave = useCallback(async (name: string) => {
    try {
      await updateUser({ userId, name })
      toast.success('User updated')
      setIsEditing(false)
      onUpdate?.()
    } catch (error) {
      toast.error('Failed to update user')
    }
  }, [updateUser, userId, onUpdate])

  // 4. Loading state
  if (user === undefined) {
    return <Skeleton className="h-24 rounded-lg" />
  }

  // 5. Not found state
  if (user === null) {
    return <div className="text-muted-foreground">User not found</div>
  }

  // 6. Render
  return (
    <div className="p-4 border rounded-lg">
      <h3 className="font-medium">{user.name}</h3>
      <p className="text-sm text-muted-foreground">{user.email}</p>
      <Button onClick={() => setIsEditing(true)}>Edit</Button>
    </div>
  )
}
```

### Feature Folder Structure

```
src/features/users/
├── components/
│   ├── UserCard.tsx
│   ├── UserList.tsx
│   ├── UserForm.tsx
│   └── UserAvatar.tsx
├── hooks/
│   └── use-user-filters.ts    # NOT Convex hooks
├── helpers/
│   └── user-helpers.ts
├── types.ts
└── index.ts                    # Public exports
```

### Lazy Loading

```typescript
import React, { Suspense } from 'react'
import { Skeleton } from '~/components/ui/skeleton'

// Lazy load heavy components
const DataGrid = React.lazy(() => import('./DataGrid'))
const Chart = React.lazy(() => import('./Chart'))

// For named exports
const UserForm = React.lazy(() =>
  import('./UserForm').then((module) => ({ default: module.UserForm }))
)

function Dashboard() {
  return (
    <div>
      <Suspense fallback={<Skeleton className="h-96 rounded" />}>
        <DataGrid />
      </Suspense>

      <Suspense fallback={<Skeleton className="h-64 rounded" />}>
        <Chart />
      </Suspense>
    </div>
  )
}
```

---

## 10.2 Loading States

### Skeleton Pattern

Match skeleton shape to actual content:

```typescript
function UserListSkeleton() {
  return (
    <div className="space-y-4">
      {/* Header */}
      <Skeleton className="h-8 w-48 rounded" />

      {/* List items */}
      {[1, 2, 3].map((i) => (
        <div key={i} className="flex items-center gap-4 p-4 border rounded">
          <Skeleton className="h-10 w-10 rounded-full" />
          <div className="space-y-2">
            <Skeleton className="h-4 w-32 rounded" />
            <Skeleton className="h-3 w-48 rounded" />
          </div>
        </div>
      ))}
    </div>
  )
}
```

### Loading Component Pattern

```typescript
function UserList() {
  const users = useQuery(api.users.list)

  if (users === undefined) {
    return <UserListSkeleton />
  }

  if (users.length === 0) {
    return (
      <div className="text-center py-8 text-muted-foreground">
        No users found
      </div>
    )
  }

  return (
    <ul className="space-y-4">
      {users.map((user) => (
        <UserCard key={user._id} user={user} />
      ))}
    </ul>
  )
}
```

### Toast Notifications

```typescript
import { toast } from 'sonner'

// Success
toast.success('User created successfully')

// Error
toast.error('Failed to create user')

// With description
toast.success('User created', {
  description: 'They will receive a welcome email shortly',
})

// Promise toast
toast.promise(createUser({ name: 'John' }), {
  loading: 'Creating user...',
  success: 'User created!',
  error: 'Failed to create user',
})
```

---

## 10.3 Styling with Tailwind + shadcn

### Tailwind Class Order

Layout → Sizing → Spacing → Borders → Colors → Effects → State

```typescript
<div className="flex items-center gap-4 p-4 border rounded-lg bg-white shadow-sm hover:shadow-md">
  <h2 className="text-lg font-semibold text-gray-900">Title</h2>
  <Button className="px-4 py-2 disabled:opacity-50">Action</Button>
</div>
```

### shadcn Component Usage

```typescript
import { Button } from '~/components/ui/button'
import { Card, CardHeader, CardTitle, CardContent } from '~/components/ui/card'
import { Input } from '~/components/ui/input'
import { Label } from '~/components/ui/label'
import {
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from '~/components/ui/dialog'

function UserDialog() {
  return (
    <Dialog>
      <DialogTrigger asChild>
        <Button>Open</Button>
      </DialogTrigger>
      <DialogContent>
        <DialogHeader>
          <DialogTitle>Edit User</DialogTitle>
        </DialogHeader>
        <div className="space-y-4">
          <div>
            <Label htmlFor="name">Name</Label>
            <Input id="name" placeholder="Enter name" />
          </div>
        </div>
      </DialogContent>
    </Dialog>
  )
}
```

### Chart Colors

Use CSS variables defined in your Tailwind config:

```typescript
const chartConfig = {
  primary: {
    label: "Primary",
    color: "var(--chart-1)",
  },
  secondary: {
    label: "Secondary",
    color: "var(--chart-2)",
  },
  tertiary: {
    label: "Tertiary",
    color: "var(--chart-3)",
  },
}

// In recharts
<Bar dataKey="value" fill="var(--chart-1)" />
<Line dataKey="trend" stroke="var(--chart-2)" />
```

---

