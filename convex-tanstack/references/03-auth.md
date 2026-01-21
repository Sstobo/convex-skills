# Part 3: Authentication

## 3.1 Better Auth + Convex Setup

### convex/auth.ts

```typescript
import { betterAuth } from 'better-auth/minimal'
import { createClient } from '@convex-dev/better-auth'
import { convex } from '@convex-dev/better-auth/plugins'
import authConfig from './auth.config'
import { components } from './_generated/api'
import { query } from './_generated/server'
import type { GenericCtx } from '@convex-dev/better-auth'
import type { DataModel } from './_generated/dataModel'

const siteUrl = process.env.SITE_URL!

// Create the Better Auth Convex client
export const authComponent = createClient<DataModel>(components.betterAuth)

// Factory function to create auth instance
export const createAuth = (ctx: GenericCtx<DataModel>) => {
  return betterAuth({
    baseURL: siteUrl,
    database: authComponent.adapter(ctx),
    emailAndPassword: {
      enabled: true,
      requireEmailVerification: false,
    },
    plugins: [
      convex({ authConfig }),
    ],
  })
}

// Query to get current authenticated user
export const getCurrentUser = query({
  args: {},
  handler: async (ctx) => {
    return await authComponent.getAuthUser(ctx)
  },
})
```

### convex/auth.config.ts

```typescript
import { getAuthConfigProvider } from '@convex-dev/better-auth/auth-config'
import type { AuthConfig } from 'convex/server'

export default {
  providers: [getAuthConfigProvider()],
} satisfies AuthConfig
```

### convex/http.ts

```typescript
import { httpRouter } from "convex/server"
import { authComponent, createAuth } from "./auth"

const http = httpRouter()

// Register Better Auth routes
authComponent.registerRoutes(http, createAuth)

// Add custom HTTP endpoints here
// http.route({
//   path: "/api/custom",
//   method: "POST",
//   handler: httpAction(async (ctx, req) => { ... }),
// })

export default http
```

### src/lib/auth-client.ts

```typescript
import { createAuthClient } from 'better-auth/react'
import { convexClient } from '@convex-dev/better-auth/client/plugins'

export const authClient = createAuthClient({
  plugins: [convexClient()],
})
```

### src/lib/auth-server.ts

```typescript
import { convexBetterAuthReactStart } from '@convex-dev/better-auth/react-start'

export const {
  handler,           // API route handler
  getToken,          // Get auth token for SSR
  fetchAuthQuery,    // Execute authenticated query
  fetchAuthMutation, // Execute authenticated mutation
  fetchAuthAction,   // Execute authenticated action
} = convexBetterAuthReactStart({
  convexUrl: process.env.VITE_CONVEX_URL!,
  convexSiteUrl: process.env.VITE_CONVEX_SITE_URL!,
})
```

---

## 3.2 API Route Handler

### src/routes/api/auth/$.ts

```typescript
import { createFileRoute } from '@tanstack/react-router'
import { handler } from '~/lib/auth-server'

export const Route = createFileRoute('/api/auth/$')({
  server: {
    handlers: {
      GET: ({ request }) => handler(request),
      POST: ({ request }) => handler(request),
    },
  },
})
```

This catch-all route delegates all `/api/auth/*` requests to Better Auth.

---

## 3.3 Authentication Flows

### Sign Up

```typescript
import { authClient } from '~/lib/auth-client'
import { useNavigate } from '@tanstack/react-router'

function SignUpForm() {
  const navigate = useNavigate()
  const [error, setError] = useState<string | null>(null)
  const [isLoading, setIsLoading] = useState(false)

  const handleSignUp = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault()
    setIsLoading(true)
    setError(null)

    const formData = new FormData(e.currentTarget)

    const result = await authClient.signUp.email({
      name: formData.get('name') as string,
      email: formData.get('email') as string,
      password: formData.get('password') as string,
    })

    if (result.error) {
      setError(result.error.message ?? 'Sign up failed')
      setIsLoading(false)
    } else {
      navigate({ to: '/dashboard' })
    }
  }

  return (
    <form onSubmit={handleSignUp}>
      <input name="name" type="text" required placeholder="Name" />
      <input name="email" type="email" required placeholder="Email" />
      <input name="password" type="password" required minLength={8} placeholder="Password" />
      {error && <p className="text-red-500">{error}</p>}
      <button type="submit" disabled={isLoading}>
        {isLoading ? 'Creating account...' : 'Sign Up'}
      </button>
    </form>
  )
}
```

### Sign In

```typescript
import { authClient } from '~/lib/auth-client'
import { useNavigate } from '@tanstack/react-router'

function SignInForm() {
  const navigate = useNavigate()
  const [error, setError] = useState<string | null>(null)
  const [isLoading, setIsLoading] = useState(false)

  const handleSignIn = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault()
    setIsLoading(true)
    setError(null)

    const formData = new FormData(e.currentTarget)

    const result = await authClient.signIn.email({
      email: formData.get('email') as string,
      password: formData.get('password') as string,
    })

    if (result.error) {
      setError(result.error.message ?? 'Sign in failed')
      setIsLoading(false)
    } else {
      navigate({ to: '/dashboard' })
    }
  }

  return (
    <form onSubmit={handleSignIn}>
      <input name="email" type="email" required placeholder="Email" />
      <input name="password" type="password" required placeholder="Password" />
      {error && <p className="text-red-500">{error}</p>}
      <button type="submit" disabled={isLoading}>
        {isLoading ? 'Signing in...' : 'Sign In'}
      </button>
    </form>
  )
}
```

### Sign Out

```typescript
import { authClient } from '~/lib/auth-client'

function SignOutButton() {
  const handleSignOut = async () => {
    await authClient.signOut({
      fetchOptions: {
        onSuccess: () => {
          // Page reload required due to expectAuth: true
          location.reload()
        },
      },
    })
  }

  return (
    <button onClick={handleSignOut}>
      Sign Out
    </button>
  )
}
```

> **Important**: When using `expectAuth: true`, you must reload the page after sign out to clear the authenticated state.

### Protected Route Pattern

```typescript
// src/routes/_app.tsx
import { createFileRoute, redirect, Outlet } from '@tanstack/react-router'

export const Route = createFileRoute('/_app')({
  beforeLoad: async ({ context, location }) => {
    if (!context.isAuthenticated) {
      throw redirect({
        to: '/login',
        search: { redirect: location.href },
      })
    }
  },
  component: () => <Outlet />,
})
```

---

## 3.4 Server-Side Authentication

### Authenticated Server Functions

```typescript
// src/routes/profile.ts
import { createServerFn } from '@tanstack/react-start'
import { fetchAuthQuery, fetchAuthMutation } from '~/lib/auth-server'
import { api } from '~/convex/_generated/api'

// Fetch user data on the server
export const getUserProfile = createServerFn({ method: 'GET' }).handler(
  async () => {
    return await fetchAuthQuery(api.users.getCurrentUser, {})
  }
)

// Update user data on the server
export const updateProfile = createServerFn({ method: 'POST' })
  .inputValidator((data: { name: string }) => data)
  .handler(async ({ data }) => {
    return await fetchAuthMutation(api.users.updateProfile, {
      name: data.name,
    })
  })
```

### Using Auth in Convex Mutations

```typescript
// convex/users.ts
import { mutation, query } from "./_generated/server"
import { v } from "convex/values"
import { authComponent } from "./auth"

export const getCurrentUser = query({
  args: {},
  handler: async (ctx) => {
    const authUser = await authComponent.getAuthUser(ctx)
    if (!authUser) return null

    // Fetch additional user data from your users table
    return await ctx.db
      .query("users")
      .withIndex("by_authId", (q) => q.eq("authId", authUser._id))
      .unique()
  },
})

export const updateProfile = mutation({
  args: {
    name: v.string(),
  },
  handler: async (ctx, args) => {
    const authUser = await authComponent.getAuthUser(ctx)
    if (!authUser) {
      throw new Error("Not authenticated")
    }

    const user = await ctx.db
      .query("users")
      .withIndex("by_authId", (q) => q.eq("authId", authUser._id))
      .unique()

    if (!user) {
      throw new Error("User not found")
    }

    await ctx.db.patch(user._id, { name: args.name })
    return { success: true }
  },
})
```

---

