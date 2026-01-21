# Part 11: Permissions

### Role Hierarchy Pattern

```typescript
// src/lib/permissions.ts
export const ROLES = {
  ADMIN: "admin",
  MANAGER: "manager",
  USER: "user",
} as const

export type Role = (typeof ROLES)[keyof typeof ROLES]

const ROLE_HIERARCHY: Role[] = [
  ROLES.USER,
  ROLES.MANAGER,
  ROLES.ADMIN,
]

export function hasRoleOrHigher(
  userRole: Role | undefined,
  requiredRole: Role
): boolean {
  if (!userRole) return false
  const userIndex = ROLE_HIERARCHY.indexOf(userRole)
  const requiredIndex = ROLE_HIERARCHY.indexOf(requiredRole)
  return userIndex >= requiredIndex
}
```

### Feature Access Pattern

```typescript
export const FEATURES = {
  VIEW_USERS: "view_users",
  EDIT_USERS: "edit_users",
  DELETE_USERS: "delete_users",
  VIEW_SETTINGS: "view_settings",
  EDIT_SETTINGS: "edit_settings",
} as const

export type Feature = (typeof FEATURES)[keyof typeof FEATURES]

const FEATURE_ACCESS: Record<Feature, Role[]> = {
  [FEATURES.VIEW_USERS]: [ROLES.USER, ROLES.MANAGER, ROLES.ADMIN],
  [FEATURES.EDIT_USERS]: [ROLES.MANAGER, ROLES.ADMIN],
  [FEATURES.DELETE_USERS]: [ROLES.ADMIN],
  [FEATURES.VIEW_SETTINGS]: [ROLES.MANAGER, ROLES.ADMIN],
  [FEATURES.EDIT_SETTINGS]: [ROLES.ADMIN],
}

export function canAccess(
  userRole: Role | undefined,
  feature: Feature
): boolean {
  if (!userRole) return false
  return FEATURE_ACCESS[feature]?.includes(userRole) ?? false
}
```

### Backend Permission Check

```typescript
// convex/lib/permissions.ts
import { QueryCtx, MutationCtx } from "../_generated/server"
import { authComponent } from "../auth"

export async function getCurrentUser(ctx: QueryCtx | MutationCtx) {
  return await authComponent.getAuthUser(ctx)
}

export async function assertAuthenticated(ctx: QueryCtx | MutationCtx) {
  const user = await getCurrentUser(ctx)
  if (!user) throw new Error("Not authenticated")
  return user
}

export async function assertRole(
  ctx: QueryCtx | MutationCtx,
  requiredRole: Role
) {
  const user = await assertAuthenticated(ctx)

  // Fetch user's role from your users table
  const userData = await ctx.db
    .query("users")
    .withIndex("by_authId", (q) => q.eq("authId", user._id))
    .unique()

  if (!userData || !hasRoleOrHigher(userData.role, requiredRole)) {
    throw new Error("Insufficient permissions")
  }

  return userData
}
```

### Component Permission Check

```typescript
import { useQuery } from 'convex-helpers/react/cache'
import { api } from '~/convex/_generated/api'
import { canAccess, FEATURES, type Role } from '~/lib/permissions'

function AdminPanel() {
  const currentUser = useQuery(api.users.getCurrentUser)

  // Loading state
  if (currentUser === undefined) {
    return <Skeleton />
  }

  // Permission check
  const userRole = currentUser?.role as Role | undefined
  if (!canAccess(userRole, FEATURES.VIEW_SETTINGS)) {
    return (
      <div className="text-center py-8">
        <h2>Access Denied</h2>
        <p>You don't have permission to view this page.</p>
      </div>
    )
  }

  return <SettingsForm />
}
```

---

