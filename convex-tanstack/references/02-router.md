# Part 2: Router & Provider Architecture

## 2.1 Router Setup

### src/router.tsx

```typescript
import { createRouter } from '@tanstack/react-router'
import { QueryClient } from '@tanstack/react-query'
import { ConvexQueryClient } from '@convex-dev/react-query'
import { ConvexProvider } from 'convex/react'
import { routeTree } from './routeTree.gen'

export function createAppRouter() {
  const convexUrl = import.meta.env.VITE_CONVEX_URL!

  // Initialize Convex Query Client
  const convexQueryClient = new ConvexQueryClient(convexUrl, {
    // Required for SSR authentication
    expectAuth: true,
  })

  // Configure React Query with Convex integration
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: {
        // Use Convex's hash function for query keys
        queryKeyHashFn: convexQueryClient.hashFn(),
        // Use Convex's query function for data fetching
        queryFn: convexQueryClient.queryFn(),
      },
    },
  })

  // Connect the clients
  convexQueryClient.connect(queryClient)

  // Create the router
  const router = createRouter({
    routeTree,
    defaultPreload: 'intent',           // Preload on hover
    defaultPreloadDelay: 50,            // Small delay before preload
    defaultViewTransition: true,        // Smooth page transitions
    scrollRestoration: true,

    // Pass clients to route context
    context: { queryClient, convexQueryClient },

    // Wrap all routes with ConvexProvider
    Wrap: ({ children }) => (
      <ConvexProvider client={convexQueryClient.convexClient}>
        {children}
      </ConvexProvider>
    ),
  })

  return router
}

// Type declaration for route context
declare module '@tanstack/react-router' {
  interface Register {
    router: ReturnType<typeof createAppRouter>
  }
}
```

### Key Configuration Options

| Option | Value | Purpose |
|--------|-------|---------|
| `expectAuth: true` | Required for SSR | Prevents queries before auth is established |
| `defaultPreload: 'intent'` | Performance | Preloads routes on link hover |
| `defaultViewTransition: true` | UX | Enables smooth page transitions |
| `scrollRestoration: true` | UX | Restores scroll position on navigation |

---

## 2.2 Root Route

### src/routes/__root.tsx

```typescript
import {
  HeadContent,
  Outlet,
  Scripts,
  createRootRouteWithContext,
  useRouteContext,
} from '@tanstack/react-router'
import { createServerFn } from '@tanstack/react-start'
import { ConvexBetterAuthProvider } from '@convex-dev/better-auth/react'
import { QueryClientProvider } from '@tanstack/react-query'
import type { ConvexQueryClient } from '@convex-dev/react-query'
import type { QueryClient } from '@tanstack/react-query'
import { authClient } from '~/lib/auth-client'
import { getToken } from '~/lib/auth-server'

// Server function to fetch auth token during SSR
const getAuth = createServerFn({ method: 'GET' }).handler(async () => {
  return await getToken()
})

// Define route context type
interface RouterContext {
  queryClient: QueryClient
  convexQueryClient: ConvexQueryClient
}

export const Route = createRootRouteWithContext<RouterContext>()({
  // Document head configuration
  head: () => ({
    meta: [
      { charSet: 'utf-8' },
      { name: 'viewport', content: 'width=device-width, initial-scale=1' },
      { title: 'My App' },
    ],
    links: [
      { rel: 'icon', href: '/favicon.ico' },
    ],
  }),

  // Runs before route loads - handles SSR auth
  beforeLoad: async (ctx) => {
    const token = await getAuth()

    // Set auth token for server-side HTTP queries
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
      <QueryClientProvider client={context.queryClient}>
        <RootDocument>
          <Outlet />
        </RootDocument>
      </QueryClientProvider>
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

### Provider Nesting Order

The order matters for proper functionality:

```
ConvexBetterAuthProvider     ← Handles auth state
  └─ QueryClientProvider     ← React Query context
      └─ RootDocument        ← HTML structure
          └─ Outlet          ← Page content
```

---

## 2.3 File-Based Routing

### Route File Structure

```
src/routes/
├── __root.tsx                    # Root layout (required)
├── index.tsx                     # Home page (/)
├── login.tsx                     # /login
├── api/
│   └── auth/
│       └── $.ts                  # /api/auth/* (catch-all)
├── _app.tsx                      # Protected layout
├── _app/
│   ├── dashboard.tsx             # /dashboard
│   ├── settings.tsx              # /settings
│   └── users/
│       ├── index.tsx             # /users
│       └── $userId.tsx           # /users/:userId
└── public/
    └── about.tsx                 # /public/about
```

### Route Naming Conventions

| Pattern | URL | Purpose |
|---------|-----|---------|
| `index.tsx` | `/` or parent path | Index route |
| `about.tsx` | `/about` | Static route |
| `$userId.tsx` | `/users/:userId` | Dynamic parameter |
| `$.tsx` | `/*` (catch-all) | Catch-all route |
| `_app.tsx` | N/A (no URL) | Layout route |
| `_app/` | Nested under `_app` | Routes using layout |

### Layout Routes

Layout routes (prefixed with `_`) don't add to the URL but wrap child routes:

```typescript
// src/routes/_app.tsx
import { createFileRoute, redirect, Outlet } from '@tanstack/react-router'

export const Route = createFileRoute('/_app')({
  // Protect all child routes
  beforeLoad: async ({ context }) => {
    if (!context.isAuthenticated) {
      throw redirect({ to: '/login' })
    }
  },
  component: AppLayout,
})

function AppLayout() {
  return (
    <div className="flex">
      <Sidebar />
      <main className="flex-1">
        <Outlet />
      </main>
    </div>
  )
}
```

### Dynamic Routes

```typescript
// src/routes/_app/users/$userId.tsx
import { createFileRoute } from '@tanstack/react-router'
import { convexQuery } from '@convex-dev/react-query'
import { useSuspenseQuery } from '@tanstack/react-query'
import { api } from '~/convex/_generated/api'

export const Route = createFileRoute('/_app/users/$userId')({
  // Preload data in loader
  loader: async ({ params, context }) => {
    await context.queryClient.ensureQueryData(
      convexQuery(api.users.getById, { userId: params.userId })
    )
  },
  component: UserPage,
})

function UserPage() {
  const { userId } = Route.useParams()
  const { data: user } = useSuspenseQuery(
    convexQuery(api.users.getById, { userId })
  )

  return <div>{user.name}</div>
}
```

---

