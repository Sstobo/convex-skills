# Better Auth + Convex + TanStack Start Setup Guide

Complete step-by-step guide for setting up Better Auth authentication with Convex and TanStack Start.

**Requirements:** Convex 1.25.0 or later

---

## Step 1: Install Packages

```bash
npm install convex@latest @convex-dev/better-auth
npm install better-auth@1.4.9 --save-exact
npm install @types/node --save-dev
```

---

## Step 2: Configure Vite for SSR

Configure Vite to bundle `@convex-dev/better-auth` during SSR to avoid module resolution issues.

**vite.config.ts:**

```typescript
export default defineConfig({
  // ...other config
  ssr: {
    noExternal: ['@convex-dev/better-auth'],
  },
})
```

---

## Step 3: Register the Component

Register the Better Auth component in your Convex project.

**convex/convex.config.ts:**

```typescript
import { defineApp } from "convex/server"
import betterAuth from "@convex-dev/better-auth/convex.config"

const app = defineApp()
app.use(betterAuth)

export default app
```

---

## Step 4: Add Convex Auth Config

Add a convex/auth.config.ts file to configure Better Auth as an authentication provider.

**convex/auth.config.ts:**

```typescript
import { getAuthConfigProvider } from '@convex-dev/better-auth/auth-config'
import type { AuthConfig } from 'convex/server'

export default {
  providers: [getAuthConfigProvider()],
} satisfies AuthConfig
```

---

## Step 5: Set Environment Variables

**Convex deployment variables** (via CLI):

```bash
# Generate a secret for encryption and hashes
npx convex env set BETTER_AUTH_SECRET=$(openssl rand -base64 32)

# Add your site URL
npx convex env set SITE_URL http://localhost:3000
```

**Local environment** (.env.local):

```bash
# Deployment used by `npx convex dev`
CONVEX_DEPLOYMENT=dev:adjective-animal-123

# Browser-accessible URLs (VITE_ prefix required)
VITE_CONVEX_URL=https://adjective-animal-123.convex.cloud
VITE_CONVEX_SITE_URL=https://adjective-animal-123.convex.site
VITE_SITE_URL=http://localhost:3000
```

---

## Step 6: Create Better Auth Instance

Create a Better Auth instance and initialize the component.

**convex/auth.ts:**

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

// The component client has methods needed for integrating Convex with Better Auth
export const authComponent = createClient<DataModel>(components.betterAuth)

export const createAuth = (ctx: GenericCtx<DataModel>) => {
  return betterAuth({
    baseURL: siteUrl,
    database: authComponent.adapter(ctx),
    emailAndPassword: {
      enabled: true,
      requireEmailVerification: false,
    },
    plugins: [
      // The Convex plugin is required for Convex compatibility
      convex({ authConfig }),
    ],
  })
}

// Query for getting the current user
export const getCurrentUser = query({
  args: {},
  handler: async (ctx) => {
    return await authComponent.getAuthUser(ctx)
  },
})
```

---

## Step 7: Create Better Auth Client

Create a Better Auth client instance for interacting with the auth server from your client.

**src/lib/auth-client.ts:**

```typescript
import { createAuthClient } from 'better-auth/react'
import { convexClient } from '@convex-dev/better-auth/client/plugins'

export const authClient = createAuthClient({
  plugins: [convexClient()],
})
```

---

## Step 8: Configure TanStack Server Utilities

Configure helper functions for authenticated SSR, server functions, and route handlers.

**src/lib/auth-server.ts:**

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

## Step 9: Mount HTTP Handlers

Register Better Auth route handlers on your Convex deployment.

**convex/http.ts:**

```typescript
import { httpRouter } from "convex/server"
import { authComponent, createAuth } from "./auth"

const http = httpRouter()
authComponent.registerRoutes(http, createAuth)

export default http
```

---

## Step 10: Set Up API Route Handler

Set up route handlers to proxy auth requests from TanStack Start to your Convex deployment.

**src/routes/api/auth/$.ts:**

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

---

## Step 11: Set Up Root Route

Wrap your application root with `ConvexBetterAuthProvider` and make auth available in loaders.

**src/routes/__root.tsx:**

```typescript
import {
  HeadContent,
  Outlet,
  Scripts,
  createRootRouteWithContext,
  useRouteContext,
} from '@tanstack/react-router'
import * as React from 'react'
import { createServerFn } from '@tanstack/react-start'
import { ConvexBetterAuthProvider } from '@convex-dev/better-auth/react'
import type { ConvexQueryClient } from '@convex-dev/react-query'
import type { QueryClient } from '@tanstack/react-query'
import { authClient } from '~/lib/auth-client'
import { getToken } from '~/lib/auth-server'

// Get auth information for SSR using available cookies
const getAuth = createServerFn({ method: 'GET' }).handler(async () => {
  return await getToken()
})

export const Route = createRootRouteWithContext<{
  queryClient: QueryClient
  convexQueryClient: ConvexQueryClient
}>()(
  head: () => ({
    meta: [
      { charSet: 'utf-8' },
      { name: 'viewport', content: 'width=device-width, initial-scale=1' },
    ],
  }),

  beforeLoad: async (ctx) => {
    const token = await getAuth()
    // All queries, mutations and actions through TanStack Query will be
    // authenticated during SSR if we have a valid token
    if (token) {
      ctx.context.convexQueryClient.serverHttpClient?.setAuth(token)
    }
    return {
      isAuthenticated: !!token,
      token,
    }
  },

  component: RootComponent,
})

function RootComponent() {
  const context = useRouteContext({ from: Route.id })

  return (
    <ConvexBetterAuthProvider
      client={context.convexQueryClient.convexClient}
      authClient={authClient}
      initialToken={context.token}
    >
      <RootDocument>
        <Outlet />
      </RootDocument>
    </ConvexBetterAuthProvider>
  )
}

function RootDocument({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <head>
        <HeadContent />
      </head>
      <body>
        {children}
        <Scripts />
      </body>
    </html>
  )
}
```

---

## Step 12: Add Router Context

Provide context from Convex to your routes with correct SSR setup.

**src/router.tsx:**

```typescript
import { createRouter } from '@tanstack/react-router'
import { QueryClient } from '@tanstack/react-query'
import { ConvexQueryClient } from '@convex-dev/react-query'
import { ConvexProvider } from 'convex/react'
import { routeTree } from './routeTree.gen'

export function createAppRouter() {
  const convexUrl = import.meta.env.VITE_CONVEX_URL!

  if (!convexUrl) {
    throw new Error('VITE_CONVEX_URL is not set')
  }

  // expectAuth: true is required for seamless SSR authentication
  const convexQueryClient = new ConvexQueryClient(convexUrl, {
    expectAuth: true,
  })

  const queryClient = new QueryClient({
    defaultOptions: {
      queries: {
        queryKeyHashFn: convexQueryClient.hashFn(),
        queryFn: convexQueryClient.queryFn(),
      },
    },
  })

  convexQueryClient.connect(queryClient)

  const router = createRouter({
    routeTree,
    defaultPreload: 'intent',
    context: { queryClient, convexQueryClient },
    scrollRestoration: true,
    Wrap: ({ children }) => (
      <ConvexProvider client={convexQueryClient.convexClient}>
        {children}
      </ConvexProvider>
    ),
  })

  return router
}
```

---

## Usage Patterns

### SSR with TanStack Query

Use `ensureQueryData` and `useSuspenseQuery` for server-side rendering:

```typescript
import { createFileRoute } from "@tanstack/react-router"
import { api } from "~/convex/_generated/api"
import { convexQuery } from "@convex-dev/react-query"
import { useSuspenseQuery } from "@tanstack/react-query"

export const Route = createFileRoute("/")({
  component: App,
  loader: async ({ context }) => {
    await Promise.all([
      context.queryClient.ensureQueryData(
        convexQuery(api.auth.getCurrentUser, {})
      ),
      // Load multiple queries in parallel if needed
    ])
  },
})
```

### Sign Up

```typescript
import { authClient } from '~/lib/auth-client'

const result = await authClient.signUp.email({
  name: 'User Name',
  email: 'user@example.com',
  password: 'password123',
})

if (result.error) {
  console.error(result.error.message)
} else {
  // Success - redirect to dashboard
}
```

### Sign In

```typescript
import { authClient } from '~/lib/auth-client'

const result = await authClient.signIn.email({
  email: 'user@example.com',
  password: 'password123',
})

if (result.error) {
  console.error(result.error.message)
} else {
  // Success - redirect to dashboard
}
```

### Sign Out with expectAuth

The `expectAuth: true` setting only has effect before the initial authentication. If a user signs out and signs back in, authenticated queries will likely be called before authentication is ready, resulting in an error.

For this reason, reload the page on sign out:

```typescript
import { authClient } from "~/lib/auth-client"

const handleSignOut = async () => {
  await authClient.signOut({
    fetchOptions: {
      onSuccess: () => {
        location.reload()
      },
    },
  })
}
```

### Protected Routes

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

### Using Better Auth API from Server

Better Auth's `auth.api` methods need to run in a Convex function. The function can then be called from server code using `fetchAuthMutation`:

**convex/users.ts:**

```typescript
import { mutation } from "./_generated/server"
import { v } from "convex/values"
import { createAuth, authComponent } from "./auth"

export const updateUserPassword = mutation({
  args: {
    currentPassword: v.string(),
    newPassword: v.string(),
  },
  handler: async (ctx, args) => {
    const { auth, headers } = await authComponent.getAuth(createAuth, ctx)
    await auth.api.changePassword({
      body: {
        currentPassword: args.currentPassword,
        newPassword: args.newPassword,
      },
      headers,
    })
  },
})
```

**src/routes/users.ts:**

```typescript
import { createServerFn } from "@tanstack/react-start"
import { fetchAuthMutation } from "~/lib/auth-server"
import { api } from "~/convex/_generated/api"

export const updatePassword = createServerFn({ method: "POST" }).handler(
  async ({ data: { currentPassword, newPassword } }) => {
    await fetchAuthMutation(api.users.updatePassword, {
      currentPassword,
      newPassword,
    })
  }
)
```

---

## Troubleshooting

### Module Resolution Issues

If you see module resolution errors during SSR, ensure `vite.config.ts` has:

```typescript
ssr: {
  noExternal: ['@convex-dev/better-auth'],
}
```

### Auth Not Working on First Load

Ensure `expectAuth: true` is set in the `ConvexQueryClient` constructor:

```typescript
const convexQueryClient = new ConvexQueryClient(convexUrl, {
  expectAuth: true,
})
```

### TypeScript Errors in convex/auth.ts

Some TypeScript errors will resolve after saving the file and running `npx convex dev` to generate types.

### Queries Failing After Sign Out

With `expectAuth: true`, queries may fail if called before re-authentication. Always reload the page on sign out to reset the client state.
