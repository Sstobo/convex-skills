# Convex Database Skills for Claude Code

A comprehensive collection of specialized skills for Claude Code that provide expert guidance on building applications with Convex. These skills draw from official Convex documentation and Convex Chef best practices to deliver production-ready patterns for real-time backend development.

## What is This?

This repository contains **17 specialized skills** that augment Claude Code with deep expertise in:
- **Core Convex**: Queries, mutations, and actions
- **Convex Agents**: Building AI-powered conversational interfaces with the @convex-dev/agent component
- **Advanced Features**: File storage, HTTP endpoints, scheduling, rate limiting, and more

Each skill provides context-aware guidance, code examples, and best practices that help you build faster and avoid common pitfalls.

## Installation

Simply copy the skill folders you need directly into your project's `.claude/skills/` directory:

```bash
# Copy all skills
cp -r convex-* /path/to/your-project/.claude/skills/

# Or copy specific skills
cp -r convex-queries /path/to/your-project/.claude/skills/
cp -r convex-agents-fundamentals /path/to/your-project/.claude/skills/
```

Once installed, Claude Code will automatically load these skills and use them when you're working with Convex code.

## Available Skills

### Core Convex Functions

| Skill | Description | Use When |
|-------|-------------|----------|
| **convex-queries** | Query function implementation, pagination, indexing, full-text search | Reading data, implementing search, optimizing queries |
| **convex-mutations** | Mutation functions, database operations (insert/patch/replace), transactions | Writing/updating data, scheduling background jobs |
| **convex-actions-general** | Actions, HTTP endpoints, validators, schemas, env vars, file storage, limits | External API calls, webhooks, schema design, configuration |

### Convex Agents Framework

The [@convex-dev/agent](https://github.com/get-convex/agent) component enables building AI-powered conversational interfaces with automatic message history, tool calling, and multi-agent orchestration.

#### Foundation

| Skill | Description | Use When |
|-------|-------------|----------|
| **convex-agents-fundamentals** | Agent setup, thread creation, basic text generation | Starting a new agent project, creating chat interfaces |
| **convex-agents-threads** | Thread management, conversation organization, metadata | Managing multi-turn conversations, per-user history |
| **convex-agents-messages** | Message storage, retrieval, role management | Saving/loading chat history, building message UIs |

#### AI Capabilities

| Skill | Description | Use When |
|-------|-------------|----------|
| **convex-agents-tools** | Tool calling, function execution, agent actions | Letting agents interact with your database or APIs |
| **convex-agents-streaming** | Real-time response streaming, UI updates | Building responsive chat interfaces with typing indicators |
| **convex-agents-rag** | Retrieval-augmented generation, embeddings, semantic search | Building knowledge bases, context-aware responses |
| **convex-agents-workflows** | Multi-step agent workflows, state machines | Complex multi-turn interactions, guided conversations |

#### Multi-Agent Systems

| Skill | Description | Use When |
|-------|-------------|----------|
| **convex-agents-human-agents** | Human-in-the-loop patterns, escalation | Building approval workflows, human oversight |
| **convex-agents-context** | Context management, conversation memory | Managing long conversations, context windows |

#### Production Features

| Skill | Description | Use When |
|-------|-------------|----------|
| **convex-agents-rate-limiting** | Rate limiting, quota management, usage controls | Preventing abuse, managing API costs |
| **convex-agents-usage-tracking** | Token counting, cost tracking, analytics | Monitoring usage, billing, optimization |
| **convex-agents-files** | File handling in conversations, attachments | Processing uploaded documents, images |
| **convex-agents-debugging** | Debugging agents, logging, error handling | Troubleshooting agent behavior, monitoring |
| **convex-agents-playground** | Testing agents, experimentation environment | Development, testing different prompts/models |

## Skill Structure

Each skill follows a consistent structure:

```
skill-name/
├── SKILL.md              # Main skill documentation with examples
├── references/           # Detailed reference documentation
│   └── *.md
├── scripts/              # Helper scripts (if applicable)
└── assets/               # Images, diagrams (if applicable)
```

## How Claude Code Uses These Skills

When you're working in a Convex project, Claude Code will:
1. **Automatically detect** when you're working with Convex code
2. **Load relevant skills** based on your task (queries, mutations, agents, etc.)
3. **Provide expert guidance** using patterns from the skill documentation
4. **Generate production-ready code** that follows Convex best practices

You don't need to explicitly invoke skills—Claude Code intelligently applies them based on context.

## Quick Start Examples

### Creating a Query with Pagination

When you ask Claude to create a paginated query, the **convex-queries** skill ensures:
- Proper use of `paginationOptsValidator`
- Index usage instead of `filter()`
- Correct ordering and result collection

```typescript
export const getMessages = query({
  args: {
    channelId: v.id("channels"),
    paginationOpts: paginationOptsValidator,
  },
  handler: async (ctx, args) => {
    return await ctx.db
      .query("messages")
      .withIndex("by_channel", (q) => q.eq("channelId", args.channelId))
      .order("desc")
      .paginate(args.paginationOpts);
  },
});
```

### Building a Chat Agent

The **convex-agents-fundamentals** skill guides you through:
- Component installation in `convex.config.ts`
- Agent instance creation with language models
- Thread and message management

```typescript
import { Agent } from "@convex-dev/agent";
import { openai } from "@ai-sdk/openai";

export const chatAgent = new Agent(components.agent, {
  name: "Support Assistant",
  languageModel: openai.chat("gpt-4o-mini"),
  instructions: "You are a helpful customer support assistant.",
});
```

### Adding Tools to an Agent

The **convex-agents-tools** skill shows you how to:
- Define tools with proper validators
- Implement tool handlers
- Attach tools to agents

```typescript
const searchTool = tool({
  description: "Search the knowledge base",
  parameters: z.object({
    query: z.string(),
  }),
  execute: async ({ query }) => {
    // Search implementation
    return results;
  },
});

export const assistantAgent = new Agent(components.agent, {
  languageModel: openai.chat("gpt-4o"),
  tools: { search: searchTool },
});
```

## Key Principles Across All Skills

These principles are reinforced throughout the skill collection:

1. **Always use validators** - Every function must have argument validators
2. **Avoid `filter()` in queries** - Use indexes with `withIndex()` instead
3. **Auth doesn't propagate** - Scheduled jobs have null auth; use internal functions
4. **Type safety** - Use `Id<"table">` and `Doc<"table">` types
5. **ASCII field names** - Object keys must be ASCII (no emoji)
6. **Return type annotations** - Add explicit returns to avoid circular references
7. **Transaction awareness** - Understand mutation transaction boundaries
8. **Respect limits** - 1 second for queries/mutations, 8192 writes, 16384 reads

## Resources

- [Convex Documentation](https://docs.convex.dev)
- [Convex Agents Component](https://github.com/get-convex/agent)
- [Claude Code Documentation](https://docs.claude.com/claude-code)
- [Convex Discord Community](https://convex.dev/community)

## Contributing

To add new skills or improve existing ones:

1. Follow the existing skill structure (SKILL.md + references/)
2. Include practical code examples
3. Reference official Convex documentation
4. Test examples in real projects
5. Keep focused on specific use cases

## License

This collection is designed for use with Claude Code and Convex. Refer to individual component licenses:
- Claude Code: [Anthropic License](https://claude.com/claude-code)
- Convex: [Convex License](https://convex.dev)

## Updates

This collection is maintained to stay current with:
- Convex platform updates
- @convex-dev/agent component changes
- Claude Code best practices
- Community feedback and patterns

---

**Ready to build with Convex?** Copy these skills to your `.claude/skills/` directory and let Claude Code guide you through building real-time, AI-powered applications.
