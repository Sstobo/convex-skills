# Part 1: Foundation

## 1.1 Overview & Philosophy

### What is Convex + TanStack Start?

**TanStack Start** is a full-stack React meta-framework built on TanStack Router and Vite. It provides server-side rendering, streaming, and type-safe server functions with deployment to any Vite-compatible hosting provider.

**Convex** is a reactive backend-as-a-service with real-time sync, automatic caching, and TypeScript-first design. Instead of REST or GraphQL, Convex uses a reactive sync engine where data changes push to clients automatically.

Together, they provide:
- **Live-updating queries** via React Query integration
- **Subscription session resumption** from SSR to live on client
- **Loader-based preloading** and prefetching
- **Consistent logical timestamp** during SSR
- **End-to-end type safety** from database to UI

### The Reactive Sync Engine Philosophy

Convex queries are not traditional API calls. They are **subscriptions**:

```
Traditional REST/GraphQL:
  Client → Request → Server → Response → Client (stale immediately)

Convex Reactive Model:
  Client ← Subscribe ← Server (pushes updates automatically)
```

**Core Principles:**

1. **Center your app on the sync engine** - The deterministic, reactive database is the single source of truth
2. **Use a query for (almost) every read** - Queries are reactive, cacheable, consistent, and resilient
3. **Keep functions tiny & fast** - Work with < a few hundred records; finish in < 100ms
4. **Prefer queries/mutations over actions** - Actions are slower, costlier, and carry fewer guarantees
5. **Minimize client-side state** - Rely on Convex's built-in caching and consistency

### Key Benefits

| Feature | Description |
|---------|-------------|
| **Real-time by default** | All queries automatically receive updates when data changes |
| **Type-safe end-to-end** | Schema generates TypeScript types for frontend and backend |
| **No cache invalidation** | Data is never stale; updates push automatically |
| **SSR support** | Server renders with consistent database snapshot |
| **Transactional mutations** | Atomic database operations with automatic retries |

---

## 1.2 Project Setup

### Quick Start (Recommended)

```bash
npm create convex@latest -- -t tanstack-start my-app
cd my-app
npm install
npm run dev
```

This scaffolds a complete project with Convex and TanStack Start pre-configured.

### Manual Setup

#### Step 1: Create TanStack Start App

```bash
npx create-start-app@latest my-app
cd my-app
```

#### Step 2: Install Dependencies

```bash
# Core Convex + TanStack Query integration
npm install convex @convex-dev/react-query @tanstack/react-query

# Better Auth for authentication
npm install better-auth@1.4.9 @convex-dev/better-auth

# Recommended utilities
npm install convex-helpers zod sonner

# Dev dependencies
npm install -D @types/node
```

#### Step 3: Initialize Convex

```bash
npx convex dev
```

This will:
- Create the `convex/` directory
- Authenticate via your browser
- Generate `.env.local` with deployment URLs
- Start the Convex dev server

### Environment Variables

**.env.local** (local development):

```bash
# Convex deployment (auto-generated)
CONVEX_DEPLOYMENT=dev:adjective-animal-123

# Browser-accessible (VITE_ prefix required)
VITE_CONVEX_URL=https://adjective-animal-123.convex.cloud
VITE_CONVEX_SITE_URL=https://adjective-animal-123.convex.site
VITE_SITE_URL=http://localhost:3000
```

**Convex environment variables** (set via CLI):

```bash
# Authentication secret (32+ characters)
npx convex env set BETTER_AUTH_SECRET=$(openssl rand -base64 32)

# Site URL for auth redirects
npx convex env set SITE_URL http://localhost:3000

# AI providers (optional)
npx convex env set OPENAI_API_KEY sk-...
npx convex env set ANTHROPIC_API_KEY sk-ant-...
```

---

## 1.3 Configuration Files

### vite.config.ts

```typescript
import { tanstackStart } from '@tanstack/react-start/plugin/vite'
import { defineConfig } from 'vite'
import tsConfigPaths from 'vite-tsconfig-paths'
import tailwindcss from '@tailwindcss/vite'
import viteReact from '@vitejs/plugin-react'

export default defineConfig({
  server: {
    port: 3000,
  },
  ssr: {
    // Critical for Better Auth SSR
    noExternal: ['@convex-dev/better-auth'],
  },
  plugins: [
    tailwindcss(),
    tsConfigPaths({
      projects: ['./tsconfig.json'],
    }),
    tanstackStart(),
    viteReact(),
  ],
})
```

### tsconfig.json

```json
{
  "include": ["**/*.ts", "**/*.tsx"],
  "exclude": ["node_modules"],
  "compilerOptions": {
    "target": "ES2022",
    "jsx": "react-jsx",
    "module": "ESNext",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],

    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "~/*": ["./src/*"]
    },

    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,

    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "verbatimModuleSyntax": true,
    "skipLibCheck": true,
    "noEmit": true,

    "esModuleInterop": true,
    "isolatedModules": true,
    "resolveJsonModule": true,
    "allowJs": true
  }
}
```

**Path Aliases**: Both `@/*` and `~/*` resolve to `src/`. Use consistently throughout your project.

### convex/convex.config.ts

```typescript
import { defineApp } from "convex/server"
import betterAuth from "@convex-dev/better-auth/convex.config"

const app = defineApp()
app.use(betterAuth)

// Optional: Add more components
// import agent from "@convex-dev/agent/convex.config"
// import rag from "@convex-dev/rag/convex.config"
// app.use(agent)
// app.use(rag)

export default app
```

### package.json Scripts

```json
{
  "scripts": {
    "dev": "concurrently \"npm:dev:convex\" \"npm:dev:web\"",
    "dev:web": "vite",
    "dev:convex": "convex dev",
    "build": "vite build && tsc --noEmit",
    "start": "node .output/server/index.mjs",
    "lint": "tsc && eslint . --max-warnings 0",
    "format": "prettier --write ."
  }
}
```

### components.json (shadcn/ui)

```json
{
  "$schema": "https://ui.shadcn.com/schema.json",
  "style": "new-york",
  "rsc": false,
  "tsx": true,
  "tailwind": {
    "config": "",
    "css": "src/styles/app.css",
    "baseColor": "neutral",
    "cssVariables": true
  },
  "iconLibrary": "lucide",
  "aliases": {
    "components": "~/components",
    "utils": "~/lib/utils",
    "ui": "~/components/ui",
    "lib": "~/lib",
    "hooks": "~/hooks"
  }
}
```

---

