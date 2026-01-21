# Part 8: Scheduling & Crons

### Schedule Future Work

```typescript
import { mutation } from "./_generated/server"
import { internal } from "./_generated/api"
import { v } from "convex/values"

export const createReminder = mutation({
  args: {
    message: v.string(),
    delayMs: v.number(),
  },
  returns: v.id("reminders"),
  handler: async (ctx, args) => {
    const reminderId = await ctx.db.insert("reminders", {
      message: args.message,
      scheduledFor: Date.now() + args.delayMs,
    })

    // Schedule action to run after delay
    await ctx.scheduler.runAfter(
      args.delayMs,
      internal.reminders.sendReminder,
      { reminderId }
    )

    return reminderId
  },
})
```

### Cron Jobs

```typescript
// convex/crons.ts
import { cronJobs } from "convex/server"
import { internal } from "./_generated/api"

const crons = cronJobs()

// Run every hour
crons.interval(
  "cleanup old sessions",
  { hours: 1 },
  internal.sessions.cleanupOld,
  {}
)

// Run at specific times (cron syntax)
crons.cron(
  "daily report",
  "0 9 * * *",  // 9 AM daily
  internal.reports.generateDaily,
  {}
)

// Run every 5 minutes
crons.interval(
  "sync external data",
  { minutes: 5 },
  internal.sync.fetchExternalData,
  {}
)

export default crons
```

### Cron Handler

```typescript
// convex/sessions.ts
import { internalMutation } from "./_generated/server"
import { v } from "convex/values"

export const cleanupOld = internalMutation({
  args: {},
  returns: v.object({ deleted: v.number() }),
  handler: async (ctx) => {
    const cutoff = Date.now() - 24 * 60 * 60 * 1000 // 24 hours ago

    const oldSessions = await ctx.db
      .query("sessions")
      .withIndex("by_lastActive", (q) => q.lt("lastActive", cutoff))
      .take(100)

    for (const session of oldSessions) {
      await ctx.db.delete(session._id)
    }

    return { deleted: oldSessions.length }
  },
})
```

---

