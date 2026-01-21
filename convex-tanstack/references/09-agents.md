# Part 9: AI & Agents

### Model Configuration

```typescript
// convex/model.ts
import { openai } from "@ai-sdk/openai"

export const chat = openai.chat("gpt-4o")
export const textEmbedding = openai.textEmbeddingModel("text-embedding-3-small")
```

### Agent Definition

```typescript
// convex/agents/myAgent.ts
import { Agent } from "@convex-dev/agent"
import { components } from "../_generated/api"
import { chat } from "../model"
import { myAgentTools } from "./tools"

export const myAgent = new Agent(components.agent, {
  name: "MyAgent",
  languageModel: chat,
  instructions: `You are a helpful assistant that...`,
  maxSteps: 10,
  tools: myAgentTools,
  contextOptions: {
    recentMessages: 20,
    excludeToolMessages: false,
  },
})
```

### Tool Definition

```typescript
// convex/agents/tools.ts
import { createTool } from "@convex-dev/agent"
import { z } from "zod"
import { internal } from "../_generated/api"

export const searchDocuments = createTool({
  description: "Search for documents matching a query",
  args: z.object({
    query: z.string().describe("The search query"),
    limit: z.number().optional().describe("Max results to return"),
  }),
  handler: async (ctx, { query, limit = 10 }) => {
    const results = await ctx.runQuery(
      internal.search.findDocuments,
      { query, limit }
    )
    return JSON.stringify(results)
  },
})

export const myAgentTools = {
  searchDocuments,
  // Add more tools...
}
```

### Streaming Chat

```typescript
// convex/agents/chat.ts
import { mutation, action } from "../_generated/server"
import { v } from "convex/values"
import { myAgent } from "./myAgent"

export const sendMessage = mutation({
  args: {
    threadId: v.optional(v.string()),
    message: v.string(),
  },
  handler: async (ctx, args) => {
    // Create or get thread
    let threadId = args.threadId
    if (!threadId) {
      const thread = await myAgent.createThread(ctx, {})
      threadId = thread.threadId
    }

    // Schedule streaming response
    await ctx.scheduler.runAfter(0, internal.agents.chat.streamResponse, {
      threadId,
      message: args.message,
    })

    return { threadId }
  },
})

export const streamResponse = internalAction({
  args: {
    threadId: v.string(),
    message: v.string(),
  },
  handler: async (ctx, args) => {
    await myAgent.continueThread(ctx, {
      threadId: args.threadId,
      content: args.message,
    })
  },
})
```

### RAG Setup

```typescript
// convex/rag/index.ts
import { Rag } from "@convex-dev/rag"
import { components } from "../_generated/api"
import { textEmbedding } from "../model"

export const rag = new Rag(components.rag, {
  embeddingModel: textEmbedding,
})

// Index documents
export const indexDocument = internalMutation({
  args: {
    documentId: v.id("documents"),
    content: v.string(),
  },
  handler: async (ctx, args) => {
    await rag.index(ctx, {
      documentId: args.documentId,
      content: args.content,
    })
  },
})

// Search documents
export const search = internalQuery({
  args: {
    query: v.string(),
    limit: v.optional(v.number()),
  },
  handler: async (ctx, args) => {
    return await rag.search(ctx, {
      query: args.query,
      limit: args.limit ?? 10,
    })
  },
})
```

---

