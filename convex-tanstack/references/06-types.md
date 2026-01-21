# Part 6: Type Safety

## 6.1 TypeScript Patterns

### Document and ID Types

```typescript
import type { Id, Doc } from "~/convex/_generated/dataModel"

// Document type
type User = Doc<"users">

// ID type
type UserId = Id<"users">

// Function with typed parameters
async function getUser(ctx: QueryCtx, userId: Id<"users">): Promise<Doc<"users"> | null> {
  return await ctx.db.get(userId)
}
```

### Type Imports

Always use `import type` for type-only imports:

```typescript
// CORRECT
import type { Id, Doc } from "~/convex/_generated/dataModel"
import type { QueryCtx, MutationCtx } from "~/convex/_generated/server"

// WRONG - Imports the value, not just the type
import { Id, Doc } from "~/convex/_generated/dataModel"
```

### Generic Type Annotations

When calling functions in the same file, add type annotations:

```typescript
export const getUser = query({
  args: { userId: v.id("users") },
  handler: async (ctx, args) => {
    return await ctx.db.get(args.userId)
  },
})

export const getUserWithPosts = query({
  args: { userId: v.id("users") },
  handler: async (ctx, args) => {
    // Add type annotation to avoid circular reference issues
    const user: Doc<"users"> | null = await ctx.runQuery(api.users.getUser, {
      userId: args.userId,
    })

    if (!user) return null

    const posts = await ctx.db
      .query("posts")
      .withIndex("by_authorId", (q) => q.eq("authorId", args.userId))
      .collect()

    return { user, posts }
  },
})
```

### as const for Literals

```typescript
// For discriminated unions
const status = "published" as const

// In objects
const config = {
  type: "email" as const,
  enabled: true,
}
```

---

## 6.2 Validators & Types

### Convex Type Mapping

| Convex Type | TypeScript | Validator | Example |
|-------------|------------|-----------|---------|
| Id | `string` | `v.id("table")` | `v.id("users")` |
| Null | `null` | `v.null()` | - |
| Int64 | `bigint` | `v.int64()` | `3n` |
| Float64 | `number` | `v.number()` | `3.14` |
| Boolean | `boolean` | `v.boolean()` | `true` |
| String | `string` | `v.string()` | `"hello"` |
| Bytes | `ArrayBuffer` | `v.bytes()` | - |
| Array | `Array` | `v.array(items)` | `v.array(v.string())` |
| Object | `Object` | `v.object({...})` | See below |
| Record | `Record` | `v.record(k, v)` | See below |

> **Deprecation Note**: `v.bigint()` is deprecated. Use `v.int64()` instead for 64-bit integers.

### Complex Validators

```typescript
import { v } from "convex/values"

// Object with specific fields
const userValidator = v.object({
  name: v.string(),
  email: v.string(),
  age: v.optional(v.number()),
})

// Union types
const statusValidator = v.union(
  v.literal("draft"),
  v.literal("published"),
  v.literal("archived")
)

// Discriminated union
const resultValidator = v.union(
  v.object({
    kind: v.literal("success"),
    data: v.any(),
  }),
  v.object({
    kind: v.literal("error"),
    message: v.string(),
  })
)

// Array of objects
const postsValidator = v.array(v.object({
  title: v.string(),
  content: v.string(),
}))

// Record (dynamic keys)
const scoresValidator = v.record(v.id("users"), v.number())
// Type: Record<Id<"users">, number>

// Optional fields
const optionalValidator = v.object({
  required: v.string(),
  optional: v.optional(v.string()),
})
```

### Function with Full Validation

```typescript
export const createPost = mutation({
  args: {
    title: v.string(),
    content: v.string(),
    tags: v.array(v.string()),
    metadata: v.optional(v.object({
      featured: v.boolean(),
      priority: v.number(),
    })),
  },
  returns: v.object({
    postId: v.id("posts"),
    createdAt: v.number(),
  }),
  handler: async (ctx, args) => {
    const postId = await ctx.db.insert("posts", {
      ...args,
      createdAt: Date.now(),
    })

    return {
      postId,
      createdAt: Date.now(),
    }
  },
})
```

---

