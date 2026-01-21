# Part 13: Quick Reference

## 13.1 Import Cheatsheet

```typescript
// ============================================
// DATA FETCHING
// ============================================

// Convex queries (cached, optimized)
import { useQuery } from 'convex-helpers/react/cache'

// Convex mutations
import { useMutation, useAction } from 'convex/react'

// React Query with Convex
import { useSuspenseQuery } from '@tanstack/react-query'
import { convexQuery } from '@convex-dev/react-query'
import { useConvexMutation } from '@convex-dev/react-query'

// API and types
import { api } from '~/convex/_generated/api'
import type { Id, Doc } from '~/convex/_generated/dataModel'

// ============================================
// BACKEND
// ============================================

// Function definitions
import { query, mutation, action } from "./_generated/server"
import { internalQuery, internalMutation, internalAction } from "./_generated/server"

// Function references
import { api, internal } from "./_generated/api"

// Validators
import { v } from "convex/values"

// HTTP
import { httpRouter } from "convex/server"
import { httpAction } from "./_generated/server"

// Pagination
import { paginationOptsValidator } from "convex/server"

// Crons
import { cronJobs } from "convex/server"

// ============================================
// AUTHENTICATION
// ============================================

// Client
import { authClient } from '~/lib/auth-client'

// Server utilities
import { getToken, fetchAuthQuery, fetchAuthMutation } from '~/lib/auth-server'

// Server function
import { createServerFn } from '@tanstack/react-start'

// ============================================
// UI COMPONENTS
// ============================================

// shadcn/ui
import { Button } from '~/components/ui/button'
import { Input } from '~/components/ui/input'
import { Card, CardHeader, CardTitle, CardContent } from '~/components/ui/card'
import { Dialog, DialogContent, DialogHeader, DialogTitle } from '~/components/ui/dialog'
import { Skeleton } from '~/components/ui/skeleton'

// Notifications
import { toast } from 'sonner'

// Utilities
import { cn } from '~/lib/utils'

// ============================================
// ROUTING
// ============================================

import { createFileRoute, redirect, Outlet, Link } from '@tanstack/react-router'
import { useNavigate, useParams, useSearch } from '@tanstack/react-router'

// ============================================
// REACT
// ============================================

import React, { useState, useCallback, useMemo, useEffect, useRef } from 'react'
import type { ReactNode, FC } from 'react'
```

---

## 13.2 Common Patterns Cheatsheet

### Skip Pattern

```typescript
// Basic skip
const user = useQuery(api.users.get, userId ? { userId } : "skip")

// Chained skip
const org = useQuery(api.orgs.get, user?.orgId ? { orgId: user.orgId } : "skip")
```

### Three-State Check

```typescript
if (data === undefined) return <Skeleton />  // Loading
if (data === null) return <NotFound />       // Not found
return <Content data={data} />               // Success
```

### Auth Check (Client)

```typescript
const { isAuthenticated } = useRouteContext({ from: Route.id })
if (!isAuthenticated) throw redirect({ to: '/login' })
```

### Auth Check (Backend)

```typescript
const user = await authComponent.getAuthUser(ctx)
if (!user) throw new Error("Not authenticated")
```

### Permission Check

```typescript
if (!canAccess(user?.role, FEATURES.EDIT_USERS)) {
  throw new Error("Insufficient permissions")
}
```

### Error Handling

```typescript
try {
  await mutation({ ... })
  toast.success('Success!')
} catch (error) {
  toast.error(error instanceof Error ? error.message : 'Operation failed')
}
```

### Index Query

```typescript
await ctx.db
  .query("posts")
  .withIndex("by_authorId_createdAt", (q) =>
    q.eq("authorId", authorId)
  )
  .order("desc")
  .take(10)
```

---

## 13.3 Anti-Patterns & Gotchas

### Query Anti-Patterns

| Wrong | Correct | Why |
|-------|---------|-----|
| `if (id) useQuery(...)` | `useQuery(..., id ? {...} : "skip")` | Hooks can't be conditional |
| `import { useQuery } from 'convex/react'` | `import { useQuery } from 'convex-helpers/react/cache'` | Better caching |
| `.filter(x => x.field === val)` | `.withIndex("by_field", q => q.eq("field", val))` | Table scan vs index |
| `data.field` without check | `if (data === undefined)` first | Loading state crash |

### Mutation Anti-Patterns

| Wrong | Correct | Why |
|-------|---------|-----|
| `setLoading(false)` only in catch | `finally { setLoading(false) }` | Always reset state |
| No error handling | `try/catch` with toast | User needs feedback |
| Trust client timestamps | Use `Date.now()` on server | Security |

### Type Anti-Patterns

| Wrong | Correct | Why |
|-------|---------|-----|
| `import { Doc } from ...` | `import type { Doc } from ...` | Type-only import |
| `count || 0` | `count ?? 0` | 0 is falsy |
| `items || []` | `items ?? []` | Empty array is truthy |

### Hook Anti-Patterns

| Wrong | Correct | Why |
|-------|---------|-----|
| `useMutation` in custom hook | `useMutation` in component | State isolation |
| `useEffect(() => { useQuery() })` | Direct in component | Breaks reactivity |
| `useCallback` for everything | Only when passed to children | Premature optimization |

### Backend Anti-Patterns

| Wrong | Correct | Why |
|-------|---------|-----|
| `await ctx.db.query("t").collect().find(...)` | Use index | Performance |
| Action with `ctx.db` | Use `ctx.runQuery/runMutation` | Actions can't access DB |
| Missing `"use node"` in action | Add directive | Node.js APIs need it |
| No return validator | Always add `returns: v.type()` | Type safety |