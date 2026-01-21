# Part 5: Convex Backend

## 5.1 Schema Design

### convex/schema.ts

```typescript
import { defineSchema, defineTable } from "convex/server"
import { v } from "convex/values"

export default defineSchema({
  // Users table with indexes
  users: defineTable({
    authId: v.string(),           // Better Auth user ID
    name: v.string(),
    email: v.string(),
    role: v.union(v.literal("admin"), v.literal("user")),
    organizationId: v.optional(v.id("organizations")),
    createdAt: v.number(),
  })
    .index("by_authId", ["authId"])
    .index("by_email", ["email"])
    .index("by_organizationId", ["organizationId"]),

  // Organizations table
  organizations: defineTable({
    name: v.string(),
    slug: v.string(),
    ownerId: v.id("users"),
    createdAt: v.number(),
  })
    .index("by_slug", ["slug"])
    .index("by_ownerId", ["ownerId"]),

  // Posts with compound index
  posts: defineTable({
    title: v.string(),
    content: v.string(),
    authorId: v.id("users"),
    organizationId: v.id("organizations"),
    status: v.union(v.literal("draft"), v.literal("published")),
    publishedAt: v.optional(v.number()),
    createdAt: v.number(),
  })
    .index("by_authorId", ["authorId"])
    .index("by_organizationId", ["organizationId"])
    .index("by_organizationId_status", ["organizationId", "status"])
    .index("by_organizationId_createdAt", ["organizationId", "createdAt"]),

  // Comments with reference
  comments: defineTable({
    postId: v.id("posts"),
    authorId: v.id("users"),
    content: v.string(),
    createdAt: v.number(),
  })
    .index("by_postId", ["postId"]),
})
```

### Index Naming Convention

Always include all index fields in the name:

```typescript
// GOOD
.index("by_organizationId_status", ["organizationId", "status"])
.index("by_userId_createdAt", ["userId", "createdAt"])

// BAD - Unclear which fields are indexed
.index("by_org", ["organizationId", "status"])
.index("status_index", ["organizationId", "status"])
```

### Compound Indexes

Query fields must match index field order:

```typescript
// Schema
.index("by_organizationId_status", ["organizationId", "status"])

// CORRECT - Fields in same order
.withIndex("by_organizationId_status", (q) =>
  q.eq("organizationId", orgId).eq("status", "published")
)

// CORRECT - Prefix query (only first field)
.withIndex("by_organizationId_status", (q) =>
  q.eq("organizationId", orgId)
)

// WRONG - Can't skip first field
.withIndex("by_organizationId_status", (q) =>
  q.eq("status", "published")  // Error: must query organizationId first
)
```

### System Fields

Every document automatically has:

```typescript
{
  _id: Id<"tableName">,        // Auto-generated document ID
  _creationTime: number,       // Unix timestamp (ms) when created
}
```

---

## 5.2 Query Functions

### Function Syntax (Required)

```typescript
import { query } from "./_generated/server"
import { v } from "convex/values"

export const listUsers = query({
  args: {
    limit: v.optional(v.number()),
  },
  returns: v.array(v.object({
    _id: v.id("users"),
    name: v.string(),
    email: v.string(),
  })),
  handler: async (ctx, args) => {
    const limit = args.limit ?? 50

    return await ctx.db
      .query("users")
      .order("desc")
      .take(limit)
  },
})
```

### Always Use Indexes

```typescript
// CORRECT - Use index
export const getByEmail = query({
  args: { email: v.string() },
  returns: v.union(
    v.object({
      _id: v.id("users"),
      name: v.string(),
      email: v.string(),
    }),
    v.null()
  ),
  handler: async (ctx, args) => {
    return await ctx.db
      .query("users")
      .withIndex("by_email", (q) => q.eq("email", args.email))
      .unique()
  },
})

// WRONG - Table scan (slow, expensive)
export const getByEmailBad = query({
  args: { email: v.string() },
  // Missing returns validator AND using table scan - don't do this!
  handler: async (ctx, args) => {
    const users = await ctx.db.query("users").collect()
    return users.find((u) => u.email === args.email)
  },
})
```

### Query Methods

| Method | Returns | Use Case |
|--------|---------|----------|
| `.unique()` | `Doc \| null` | Exactly 0 or 1 result expected (throws if > 1) |
| `.first()` | `Doc \| null` | Get first matching document |
| `.collect()` | `Doc[]` | Get all matching documents |
| `.take(n)` | `Doc[]` | Get up to n documents |

### Ordering

```typescript
// Default order: ascending by _creationTime
const posts = await ctx.db.query("posts").collect()

// Explicit descending order (newest first)
const posts = await ctx.db
  .query("posts")
  .order("desc")
  .collect()

// Order by index field
const posts = await ctx.db
  .query("posts")
  .withIndex("by_organizationId_createdAt", (q) =>
    q.eq("organizationId", orgId)
  )
  .order("desc")  // Orders by createdAt (last index field)
  .collect()
```

### Pagination

```typescript
import { query } from "./_generated/server"
import { v } from "convex/values"
import { paginationOptsValidator } from "convex/server"

export const listPaginated = query({
  args: {
    paginationOpts: paginationOptsValidator,
    organizationId: v.id("organizations"),
  },
  handler: async (ctx, args) => {
    return await ctx.db
      .query("posts")
      .withIndex("by_organizationId", (q) =>
        q.eq("organizationId", args.organizationId)
      )
      .order("desc")
      .paginate(args.paginationOpts)
  },
})
```

**`paginationOpts` structure:**
- `numItems`: Maximum documents to return (`v.number()`)
- `cursor`: Cursor for next page (`v.union(v.string(), v.null())`)

**Returns:**
- `page`: Array of documents fetched
- `isDone`: Boolean indicating if this is the last page
- `continueCursor`: String cursor for fetching the next page

### Full Text Search

For text search, use search indexes instead of regular indexes:

**Schema Definition:**

```typescript
// convex/schema.ts
import { defineSchema, defineTable } from "convex/server"
import { v } from "convex/values"

export default defineSchema({
  posts: defineTable({
    title: v.string(),
    content: v.string(),
    authorId: v.id("users"),
  })
    .index("by_authorId", ["authorId"])
    .searchIndex("search_content", {
      searchField: "content",
      filterFields: ["authorId"],
    }),
})
```

**Query with Search:**

```typescript
export const searchPosts = query({
  args: {
    query: v.string(),
    authorId: v.optional(v.id("users")),
  },
  returns: v.array(v.object({
    _id: v.id("posts"),
    _creationTime: v.number(),
    title: v.string(),
    content: v.string(),
    authorId: v.id("users"),
  })),
  handler: async (ctx, args) => {
    let searchQuery = ctx.db
      .query("posts")
      .withSearchIndex("search_content", (q) =>
        q.search("content", args.query)
      )

    // Optional filter
    if (args.authorId) {
      searchQuery = ctx.db
        .query("posts")
        .withSearchIndex("search_content", (q) =>
          q.search("content", args.query).eq("authorId", args.authorId)
        )
    }

    return await searchQuery.take(10)
  },
})
```

> **Note**: Search indexes are different from regular indexes. Use `.withSearchIndex()` for text search and `.withIndex()` for exact matches.

### Async Iteration (Large Datasets)

For processing large result sets efficiently, use async iteration instead of `.collect()`:

```typescript
export const processAllPosts = internalMutation({
  args: { authorId: v.id("users") },
  returns: v.object({ processed: v.number() }),
  handler: async (ctx, args) => {
    let processed = 0

    // Async iterate instead of collecting all at once
    for await (const post of ctx.db
      .query("posts")
      .withIndex("by_authorId", (q) => q.eq("authorId", args.authorId))
    ) {
      // Process each post without loading all into memory
      await ctx.db.patch(post._id, { processed: true })
      processed++
    }

    return { processed }
  },
})
```

| Method | Use Case | Memory |
|--------|----------|--------|
| `.collect()` | Small result sets (< 100 docs) | All in memory |
| `.take(n)` | Fixed limit needed | Limited to n |
| `for await` | Large/unknown result sets | One at a time |

---

## 5.3 Mutation Functions

### Basic Mutation

```typescript
import { mutation } from "./_generated/server"
import { v } from "convex/values"

export const createPost = mutation({
  args: {
    title: v.string(),
    content: v.string(),
    organizationId: v.id("organizations"),
  },
  returns: v.id("posts"),
  handler: async (ctx, args) => {
    // Auth check
    const user = await getCurrentUser(ctx)
    if (!user) throw new Error("Not authenticated")

    // Insert document
    const postId = await ctx.db.insert("posts", {
      title: args.title,
      content: args.content,
      authorId: user._id,
      organizationId: args.organizationId,
      status: "draft",
      createdAt: Date.now(),
    })

    return postId
  },
})
```

### Update with Patch

```typescript
export const updatePost = mutation({
  args: {
    postId: v.id("posts"),
    title: v.optional(v.string()),
    content: v.optional(v.string()),
  },
  returns: v.null(),
  handler: async (ctx, args) => {
    const { postId, ...updates } = args

    // Get existing document
    const post = await ctx.db.get(postId)
    if (!post) throw new Error("Post not found")

    // Remove undefined values
    const cleanUpdates = Object.fromEntries(
      Object.entries(updates).filter(([_, v]) => v !== undefined)
    )

    // Patch document (shallow merge)
    await ctx.db.patch(postId, cleanUpdates)
    return null
  },
})
```

### Replace Document

```typescript
export const replacePost = mutation({
  args: {
    postId: v.id("posts"),
    title: v.string(),
    content: v.string(),
    status: v.union(v.literal("draft"), v.literal("published")),
  },
  returns: v.null(),
  handler: async (ctx, args) => {
    const { postId, ...newData } = args

    const existing = await ctx.db.get(postId)
    if (!existing) throw new Error("Post not found")

    // Replace entire document (except _id and _creationTime)
    await ctx.db.replace(postId, {
      ...newData,
      authorId: existing.authorId,
      organizationId: existing.organizationId,
      createdAt: existing.createdAt,
    })
    return null
  },
})
```

### Delete Document

```typescript
export const deletePost = mutation({
  args: { postId: v.id("posts") },
  returns: v.null(),
  handler: async (ctx, args) => {
    const post = await ctx.db.get(args.postId)
    if (!post) throw new Error("Post not found")

    // Delete related comments first
    const comments = await ctx.db
      .query("comments")
      .withIndex("by_postId", (q) => q.eq("postId", args.postId))
      .collect()

    for (const comment of comments) {
      await ctx.db.delete(comment._id)
    }

    // Delete the post
    await ctx.db.delete(args.postId)
    return null
  },
})
```

### ID Generation Pattern

For application-level IDs (not Convex `_id`):

```typescript
function generateId(prefix: string): string {
  const timestamp = Date.now()
  const random = Math.random().toString(36).substring(2, 9)
  return `${prefix}_${timestamp}_${random}`
}

export const createOrder = mutation({
  args: { /* ... */ },
  returns: v.string(),
  handler: async (ctx, args) => {
    const orderId = generateId("ord")  // e.g., "ord_1703001234567_abc123"

    await ctx.db.insert("orders", {
      orderId,  // Application-level ID
      // ... other fields
    })

    return orderId
  },
})
```

### Idempotency Pattern

```typescript
export const processPayment = mutation({
  args: {
    paymentId: v.string(),
    amount: v.number(),
  },
  returns: v.object({
    success: v.boolean(),
    paymentId: v.string(),
  }),
  handler: async (ctx, args) => {
    // Check if already processed
    const existing = await ctx.db
      .query("payments")
      .withIndex("by_paymentId", (q) => q.eq("paymentId", args.paymentId))
      .first()

    if (existing) {
      // Already processed, return existing result
      return { success: true, paymentId: existing.paymentId }
    }

    // Process payment
    await ctx.db.insert("payments", {
      paymentId: args.paymentId,
      amount: args.amount,
      processedAt: Date.now(),
    })

    return { success: true, paymentId: args.paymentId }
  },
})
```

---

## 5.4 Action Functions

Actions are for external API calls and side effects. They **cannot access the database directly**.

### Basic Action

```typescript
"use node"  // Required for Node.js APIs

import { action } from "./_generated/server"
import { v } from "convex/values"

export const sendEmail = action({
  args: {
    to: v.string(),
    subject: v.string(),
    body: v.string(),
  },
  returns: v.object({
    success: v.boolean(),
    messageId: v.optional(v.string()),
  }),
  handler: async (ctx, args) => {
    // Call external API
    const response = await fetch("https://api.resend.com/emails", {
      method: "POST",
      headers: {
        "Authorization": `Bearer ${process.env.RESEND_API_KEY}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        from: "noreply@example.com",
        to: args.to,
        subject: args.subject,
        html: args.body,
      }),
    })

    const data = await response.json()

    return {
      success: response.ok,
      messageId: data.id,
    }
  },
})
```

### Action Calling Query/Mutation

```typescript
"use node"

import { action } from "./_generated/server"
import { internal } from "./_generated/api"
import { v } from "convex/values"

export const processOrder = action({
  args: { orderId: v.string() },
  returns: v.object({ success: v.boolean() }),
  handler: async (ctx, args) => {
    // Read data via internal query
    const order = await ctx.runQuery(internal.orders.getById, {
      orderId: args.orderId,
    })

    if (!order) throw new Error("Order not found")

    // Call external payment API
    const paymentResult = await processPaymentExternal(order)

    // Write result via internal mutation
    await ctx.runMutation(internal.orders.updateStatus, {
      orderId: args.orderId,
      status: paymentResult.success ? "paid" : "failed",
      transactionId: paymentResult.transactionId,
    })

    return { success: paymentResult.success }
  },
})
```

### Scheduling from Mutations

```typescript
import { mutation } from "./_generated/server"
import { internal } from "./_generated/api"
import { v } from "convex/values"

export const createOrder = mutation({
  args: { /* ... */ },
  returns: v.id("orders"),
  handler: async (ctx, args) => {
    const orderId = await ctx.db.insert("orders", { /* ... */ })

    // Schedule action to run after mutation commits
    await ctx.scheduler.runAfter(0, internal.orders.sendConfirmationEmail, {
      orderId,
    })

    return orderId
  },
})
```

---

## 5.5 Internal Functions

Internal functions are private and cannot be called from the client.

```typescript
import { internalQuery, internalMutation, internalAction } from "./_generated/server"
import { v } from "convex/values"

// Internal query - called by other functions
export const getByIdInternal = internalQuery({
  args: { orderId: v.string() },
  returns: v.union(
    v.object({
      _id: v.id("orders"),
      _creationTime: v.number(),
      orderId: v.string(),
      status: v.string(),
    }),
    v.null()
  ),
  handler: async (ctx, args) => {
    return await ctx.db
      .query("orders")
      .withIndex("by_orderId", (q) => q.eq("orderId", args.orderId))
      .unique()
  },
})

// Internal mutation - called by actions
export const updateStatusInternal = internalMutation({
  args: {
    orderId: v.string(),
    status: v.string(),
  },
  returns: v.null(),
  handler: async (ctx, args) => {
    const order = await ctx.db
      .query("orders")
      .withIndex("by_orderId", (q) => q.eq("orderId", args.orderId))
      .unique()

    if (order) {
      await ctx.db.patch(order._id, { status: args.status })
    }
    return null
  },
})

// Internal action - for background processing
export const processInBackground = internalAction({
  args: { data: v.any() },
  returns: v.null(),
  handler: async (ctx, args) => {
    // Process data...
    return null
  },
})
```

### Function References

```typescript
// Public functions use `api`
import { api } from "./_generated/api"
await ctx.runQuery(api.users.getById, { userId })

// Internal functions use `internal`
import { internal } from "./_generated/api"
await ctx.runQuery(internal.users.getByIdInternal, { userId })
```

---

## 5.6 HTTP Endpoints

### Basic HTTP Endpoint

```typescript
// convex/http.ts
import { httpRouter } from "convex/server"
import { httpAction } from "./_generated/server"

const http = httpRouter()

// Simple endpoint
http.route({
  path: "/api/health",
  method: "GET",
  handler: httpAction(async (ctx, request) => {
    return new Response(JSON.stringify({ status: "ok" }), {
      status: 200,
      headers: { "Content-Type": "application/json" },
    })
  }),
})

// POST with body
http.route({
  path: "/api/webhook",
  method: "POST",
  handler: httpAction(async (ctx, request) => {
    const body = await request.json()

    // Process webhook...
    await ctx.runMutation(internal.webhooks.process, { data: body })

    return new Response(null, { status: 200 })
  }),
})

export default http
```

### HTTP Endpoint with Authentication

```typescript
http.route({
  path: "/api/protected",
  method: "GET",
  handler: httpAction(async (ctx, request) => {
    const authHeader = request.headers.get("Authorization")

    if (!authHeader?.startsWith("Bearer ")) {
      return new Response("Unauthorized", { status: 401 })
    }

    const token = authHeader.substring(7)
    // Validate token...

    return new Response(JSON.stringify({ data: "protected" }), {
      status: 200,
      headers: { "Content-Type": "application/json" },
    })
  }),
})
```

---

