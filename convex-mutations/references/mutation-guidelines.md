# Convex Mutation Guidelines

## Function Definition Syntax

Mutation functions use the new function syntax. Example:

```ts
import { mutation } from "./_generated/server";
import { v } from "convex/values";
export const f = mutation({
  args: {},
  handler: async (ctx, args) => {
    // Function body
  },
});
```

## Mutation Registration

- Use `mutation` to register public mutation functions. These functions are part of the public API and are exposed to the public Internet.
- Use `internalMutation` to register internal (private) mutation functions. These functions are private and aren't part of an app's API. They can only be called by other Convex functions.
- These functions are always imported from `./_generated/server`.
- ALWAYS include argument validators for all mutation functions (both `mutation` and `internalMutation`).
- If the JavaScript implementation of a Convex mutation doesn't have a return value, it implicitly returns `null`.

## Argument Validators

Example of a mutation with argument validators:

```ts
import { mutation } from "./_generated/server";
import { v } from "convex/values";

export default mutation({
  args: {
    simpleArray: v.array(v.union(v.string(), v.number())),
  },
  handler: async (ctx, args) => {
    //...
  },
});
```

NEVER use return validators when getting started writing an app. For example:

```ts
import { mutation } from "./_generated/server";
import { v } from "convex/values";

export default mutation({
  args: {
    simpleArray: v.array(v.union(v.string(), v.number())),
  },
  // Do NOT include a return validator with the `returns` field.
  // returns: v.number(),
  handler: async (ctx, args) => {
    //...
    return 100;
  },
});
```

## Function Calling

- Use `ctx.runQuery` to call a query from a mutation.
- Use `ctx.runMutation` to call a mutation from a mutation.
- Use `ctx.runMutation` to call a mutation from an action (to write to the database from an action).
- When using `ctx.runMutation` to call a function in the same file, specify a type annotation on the return value to work around TypeScript circularity limitations.
- All of these calls take in a `FunctionReference`. Do NOT try to pass the callee function directly into one of these calls.
- Try to use as few calls from mutations to queries and other mutations as possible. Mutations are transactions, so splitting logic up into multiple calls introduces the risk of race conditions.

## Function References

- ALWAYS use the `api` object defined by the framework in `convex/_generated/api.ts` to call public mutations registered with `mutation`. You must import the `api` object in the same file when using it and it looks like:

```ts
import { api } from "./_generated/api";
```

- ALWAYS use the `internal` object defined by the framework in `convex/_generated/api.ts` to call internal (or private) mutations registered with `internalMutation`. You must import the `internal` object in the same file when using it and it looks like:

```ts
import { internal } from "./_generated/api";
```

- Convex uses file-based routing, so a public mutation defined in `convex/example.ts` named `f` has a function reference of `api.example.f`.
- A private mutation defined in `convex/example.ts` named `g` has a function reference of `internal.example.g`.
- Mutations can also be registered within directories nested within the `convex/` folder. For example, a public mutation `h` defined in `convex/messages/access.ts` has a function reference of `api.messages.access.h`.

## Mutation Guidelines

- Use `ctx.db.replace` to fully replace an existing document. This method will throw an error if the document does not exist.
- Use `ctx.db.patch` to shallow merge updates into an existing document. This method will throw an error if the document does not exist.

## Scheduling from Mutations

You can schedule a mutation or action to run in the future by calling `ctx.scheduler.runAfter(delay, functionReference, args)` from a mutation. Enqueuing a job to the scheduler is transactional from within a mutation.

You MUST use a function reference for the first argument to `runAfter`, not a string or the function itself.

Auth state does not propagate to scheduled jobs, so `getAuthUserId()` and `ctx.getUserIdentity()` will ALWAYS return `null` from within a scheduled job. Prefer using internal, privileged functions for scheduled jobs that don't need to do access checks.

Scheduled jobs should be used sparingly and never called in a tight loop. Scheduled functions should not be scheduled more than once every 10 seconds. Especially in things like a game simulation or something similar that needs many updates in a short period of time.

## Mutation Limits

- Mutations can take in at most 8 MiB of data as arguments.
- Mutations can return at most 8 MiB of data as their return value.
- Mutations can read up to 8MiB of data from the database.
- Mutations can read up to 16384 documents from the database.
- Mutations can write up to 8MiB of data to the database.
- Mutations can write up to 8192 documents to the database.
- Mutations can execute for at most 1 second.

IMPORTANT: Hitting any of these limits will cause a function call to fail with an error. You MUST design your application to avoid hitting these limits. For example, if you are building a stock ticker app, you can't store a database record for each stock ticker's price at a point in time. Instead, download the data as JSON, save it to file storage, and have the app download the JSON file into the browser and render it client-side.

## Example: Sending a Message and Scheduling AI Response

```ts
import { mutation, internalAction } from "./_generated/server";
import { v } from "convex/values";
import { internal } from "./_generated/api";

export const sendMessage = mutation({
  args: {
    channelId: v.id("channels"),
    authorId: v.id("users"),
    content: v.string(),
  },
  handler: async (ctx, args) => {
    const channel = await ctx.db.get(args.channelId);
    if (!channel) {
      throw new Error("Channel not found");
    }
    const user = await ctx.db.get(args.authorId);
    if (!user) {
      throw new Error("User not found");
    }
    await ctx.db.insert("messages", {
      channelId: args.channelId,
      authorId: args.authorId,
      content: args.content,
    });
    await ctx.scheduler.runAfter(0, internal.functions.generateResponse, {
      channelId: args.channelId,
    });
    return null;
  },
});

export const writeAgentResponse = internalMutation({
  args: {
    channelId: v.id("channels"),
    content: v.string(),
  },
  handler: async (ctx, args) => {
    await ctx.db.insert("messages", {
      channelId: args.channelId,
      content: args.content,
    });
    return null;
  },
});
```
