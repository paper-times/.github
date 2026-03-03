

### Metadata

> **Description:** Guidelines and best practices for building Convex projects, including database schema design, queries, mutations, and real-world examples.
> **Target Files:** `**/*.ts`, `**/*.tsx`, `**/*.js`, `**/*.jsx`

---

# Convex Guidelines

## 1. Function Guidelines

### New Function Syntax

> [!IMPORTANT]
> **ALWAYS** use the new function syntax for Convex functions to ensure better type inference and consistency.

```typescript
import { query } from "./_generated/server";
import { v } from "convex/values";

export const f = query({
  args: {},
  handler: async (ctx, args) => {
    // Function body
  },
});

```

### HTTP Endpoint Syntax

HTTP endpoints are defined in `convex/http.ts` and require the `httpAction` decorator. Endpoints are registered at the **exact** path specified.

```typescript
import { httpRouter } from "convex/server";
import { httpAction } from "./_generated/server";

const http = httpRouter();

http.route({
  path: "/echo",
  method: "POST",
  handler: httpAction(async (ctx, req) => {
    const body = await req.bytes();
    return new Response(body, { status: 200 });
  }),
});

```

### Validators

Validators ensure data integrity at the entry point of your functions.

**Array Validator Example:**

```typescript
import { mutation } from "./_generated/server";
import { v } from "convex/values";

export default mutation({
  args: {
    simpleArray: v.array(v.union(v.string(), v.number())),
  },
  handler: async (ctx, args) => { /* ... */ },
});

```

**Discriminated Union Schema Example:**

```typescript
import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  results: defineTable(
    v.union(
      v.object({
        kind: v.literal("error"),
        errorMessage: v.string(),
      }),
      v.object({
        kind: v.literal("success"),
        value: v.number(),
      }),
    ),
  )
});

```

### Type Mapping Table

| Convex Type | TS/JS Type | Validator | Notes |
| --- | --- | --- | --- |
| **Id** | `string` | `v.id(table)` | Unique document identifier. |
| **Null** | `null` | `v.null()` | `undefined` is not valid; use `null`. |
| **Int64** | `bigint` | `v.int64()` | Supports -2^63 to 2^63-1. |
| **Float64** | `number` | `v.number()` | IEEE-754 double precision. |
| **Boolean** | `boolean` | `v.boolean()` | Standard true/false. |
| **String** | `string` | `v.string()` | UTF-8, max 1MB. |
| **Bytes** | `ArrayBuffer` | `v.bytes()` | Binary data, max 1MB. |
| **Array** | `Array` | `v.array(v)` | Max 8192 values. |
| **Object** | `Object` | `v.object({...})` | Plain objects only; max 1024 entries. |
| **Record** | `Record` | `v.record(k, v)` | Runtime objects with dynamic keys. |

---

## 2. Function Registration & Calling

### Visibility Rules

* **Internal Functions:** Use `internalQuery`, `internalMutation`, `internalAction`. These are private and cannot be called from the Internet.
* **Public Functions:** Use `query`, `mutation`, `action`. These are exposed via the `api` object.

### Calling Conventions

* **Query/Mutation:** Use `ctx.runQuery` or `ctx.runMutation`.
* **Actions:** Use `ctx.runAction`.
* **Warning:** Only call an action from another action if you must cross runtimes (V8 to Node). Otherwise, use shared helper functions.

> [!TIP]
> To avoid TypeScript circularity issues when calling a function in the same file, use an explicit type annotation on the result:
> `const result: string = await ctx.runQuery(api.example.f, { name: "Bob" });`

---

## 3. Pagination & Search

### Paginated Queries

```typescript
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

### Full-Text Search

```typescript
const messages = await ctx.db
  .query("messages")
  .withSearchIndex("search_body", (q) =>
    q.search("body", "hello hi").eq("channel", "#general"),
  )
  .take(10);

```

---

## 4. Database Mutations

* **Replace:** `await ctx.db.replace(id, { ... })` — Fully overwrites a document.
* **Patch:** `await ctx.db.patch(id, { field: value })` — Shallow merges updates.

---

## 5. Action Guidelines (Node.js)

* Include `"use node";` at the top if using Node.js built-ins.
* **NEVER** use `ctx.db` inside an action (actions cannot access the database directly).
* **NEVER** mix `"use node";` with queries or mutations in the same file.

---

## 6. Scheduling & Crons

Always use the `crons` object and export it as default from `convex/crons.ts`.

```typescript
const crons = cronJobs();
crons.interval("cleanup", { hours: 2 }, internal.crons.empty, {});
export default crons;

```

---

## 7. File Storage

Convex stores items as `Blob` objects.

* **Get URL:** `ctx.storage.getUrl(fileId)`
* **Get Metadata:** Query the `_storage` system table instead of using the deprecated `getMetadata` method.

```typescript
const metadata = await ctx.db.system.get("_storage", args.fileId);

```

---

## 🏗️ Example: Chat App Implementation

### Schema (`convex/schema.ts`)

```typescript
import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  channels: defineTable({ name: v.string() }),
  users: defineTable({ name: v.string() }),
  messages: defineTable({
    channelId: v.id("channels"),
    authorId: v.optional(v.id("users")),
    content: v.string(),
  }).index("by_channel", ["channelId"]),
});

```

### Business Logic (`convex/index.ts`)

```typescript
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
    await ctx.db.insert("messages", {
      channelId: args.channelId,
      authorId: args.authorId,
      content: args.content,
    });
    
    // Schedule AI response asynchronously
    await ctx.scheduler.runAfter(0, internal.index.generateResponse, {
      channelId: args.channelId,
    });
  },
});

```
