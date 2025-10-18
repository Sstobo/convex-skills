# Convex Query Guidelines

## Function Definition Syntax

Query functions use the new function syntax. Example:

```ts
import { query } from "./_generated/server";
import { v } from "convex/values";
export const f = query({
  args: {},
  handler: async (ctx, args) => {
    // Function body
  },
});
```

## Query Registration

- Use `query` to register public query functions. These functions are part of the public API and are exposed to the public Internet.
- Use `internalQuery` to register internal (private) query functions. These functions are private and aren't part of an app's API. They can only be called by other Convex functions.
- These functions are always imported from `./_generated/server`.
- ALWAYS include argument validators for all query functions (both `query` and `internalQuery`).
- If the JavaScript implementation of a Convex query doesn't have a return value, it implicitly returns `null`.

## Argument Validators

Example of a query with argument validators:

```ts
import { query } from "./_generated/server";
import { v } from "convex/values";

export default query({
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
import { query } from "./_generated/server";
import { v } from "convex/values";

export default query({
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

- Use `ctx.runQuery` to call a query from a query, mutation, or action.
- When using `ctx.runQuery` to call a function in the same file, specify a type annotation on the return value to work around TypeScript circularity limitations. For example:

```ts
export const f = query({
  args: { name: v.string() },
  handler: async (ctx, args) => {
    return "Hello " + args.name;
  },
});

export const g = query({
  args: {},
  handler: async (ctx, args) => {
    const result: string = await ctx.runQuery(api.example.f, { name: "Bob" });
    return null;
  },
});
```

- All of these calls take in a `FunctionReference`. Do NOT try to pass the callee function directly into one of these calls.
- Try to use as few calls from actions to queries as possible. Queries are transactions, so splitting logic up into multiple calls introduces the risk of race conditions.

## Function References

- ALWAYS use the `api` object defined by the framework in `convex/_generated/api.ts` to call public queries registered with `query`. You must import the `api` object in the same file when using it and it looks like:

```ts
import { api } from "./_generated/api";
```

- ALWAYS use the `internal` object defined by the framework in `convex/_generated/api.ts` to call internal (or private) queries registered with `internalQuery`. You must import the `internal` object in the same file when using it and it looks like:

```ts
import { internal } from "./_generated/api";
```

- Convex uses file-based routing, so a public query defined in `convex/example.ts` named `f` has a function reference of `api.example.f`.
- A private query defined in `convex/example.ts` named `g` has a function reference of `internal.example.g`.
- Queries can also be registered within directories nested within the `convex/` folder. For example, a public query `h` defined in `convex/messages/access.ts` has a function reference of `api.messages.access.h`.

## Query Guidelines

- Do NOT use `filter` in queries. Instead, define an index in the schema and use `withIndex` instead.
- Convex queries do NOT support `.delete()`. Instead, `.collect()` the results, iterate over them, and call `ctx.db.delete(row._id)` on each result.
- Use `.unique()` to get a single document from a query. This method will throw an error if there are multiple documents that match the query.
- When using async iteration, don't use `.collect()` or `.take(n)` on the result of a query. Instead, use the `for await (const row of query)` syntax.

## Ordering

- By default Convex always returns documents in ascending `_creationTime` order.
- You can use `.order('asc')` or `.order('desc')` to pick whether a query is in ascending or descending order. If the order isn't specified, it defaults to ascending.
- Document queries that use indexes will be ordered based on the columns in the index and can avoid slow table scans.

## Pagination

- Paginated queries are queries that return a list of results in incremental pages.
- Define pagination using the following syntax:

```ts
import { v } from "convex/values";
import { query, mutation } from "./_generated/server";
import { paginationOptsValidator } from "convex/server";

export const listWithExtraArg = query({
  args: { paginationOpts: paginationOptsValidator, author: v.string() },
  handler: async (ctx, args) => {
    return await ctx.db
      .query("messages")
      .withIndex("by_author", (q) => q.eq("author", args.author))
      .order("desc")
      .paginate(args.paginationOpts);
  },
});
```

Note: `paginationOpts` is an object with the following properties:
- `numItems`: the maximum number of documents to return (the validator is `v.number()`)
- `cursor`: the cursor to use to fetch the next page of documents (the validator is `v.union(v.string(), v.null())`)

- A query that ends in `.paginate()` returns an object that has the following properties:
  - page (contains an array of documents that you fetches)
  - isDone (a boolean that represents whether or not this is the last page of documents)
  - continueCursor (a string that represents the cursor to use to fetch the next page of documents)

## Full Text Search - Querying

- A query for "10 messages in channel '#general' that best match the query 'hello hi' in their body" would look like:

```ts
const messages = await ctx.db
  .query("messages")
  .withSearchIndex("search_body", (q) =>
    q.search("body", "hello hi").eq("channel", "#general"),
  )
  .take(10);
```

## Index Definitions (For Queries)

- Index names must be unique within a table.
- The system provides two built-in indexes: "by_id" and "by_creation_time." Never add these to the schema definition of a table! They're automatic and adding them to will be an error. You cannot use either of these names for your own indexes.
- Convex automatically includes `_creationTime` as the final column in all indexes.
- Do NOT under any circumstances include `_creationTime` as the last column in any index you define. This will result in an error. `.index("by_author_and_creation_time", ["author", "_creationTime"])` is ALWAYS wrong.
- Always include all index fields in the index name. For example, if an index is defined as `["field1", "field2"]`, the index name should be "by_field1_and_field2".
- Index fields must be queried in the same order they are defined. If you want to be able to query by "field1" then "field2" and by "field2" then "field1", you must create separate indexes.
- Index definitions MUST be nonempty. `.index("by_creation_time", [])` is ALWAYS wrong.

Here's an example of correctly using the built-in `by_creation_time` index:

Path: `convex/schema.ts`
```ts
import { defineSchema } from "convex/server";

export default defineSchema({
  // IMPORTANT: No explicit `.index("by_creation_time", ["_creationTime"]) ` is needed.
  messages: defineTable({
    name: v.string(),
    body: v.string(),
  })
    // IMPORTANT: This index sorts by `(name, _creationTime)`.
    .index("by_name", ["name"]),
});
```

Path: `convex/messages.ts`
```ts
import { query } from "./_generated/server";

export const exampleQuery = query({
  args: {},
  handler: async (ctx) => {
    // This is automatically in ascending `_creationTime` order.
    const recentMessages = await ctx.db.query("messages")
      .withIndex("by_creation_time", (q) => q.gt("_creationTime", Date.now() - 60 * 60 * 1000))
      .collect();

    // This is automatically in `_creationTime` order.
    const allMessages = await ctx.db.query("messages").order("desc").collect();

    // This query uses the index to filter by the name field and then implicitly
    // orders by `_creationTime`.
    const byName = await ctx.db.query("messages")
      .withIndex("by_name", (q) => q.eq("name", "Alice"))
      .order("asc")
      .collect();
  },
});
```

## Query Limits

- Queries can take in at most 8 MiB of data as arguments.
- Queries can return at most 8 MiB of data as their return value.
- Queries can read up to 8MiB of data from the database.
- Queries can read up to 16384 documents from the database.
- Queries can execute for at most 1 second.

IMPORTANT: Hitting any of these limits will cause a function call to fail with an error. You MUST design your application to avoid hitting these limits.
