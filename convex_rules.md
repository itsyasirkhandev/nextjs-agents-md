# ⚡ Convex Rules & Best Practices
# Version: 1.0.0
# Last Updated: 2026-02-21
# Convex Package: ^1.31.0+

---

## 📑 Table of Contents

1.  [Philosophy & Core Principles](#1-philosophy--core-principles)
2.  [Project File Structure](#2-project-file-structure)
3.  [Function Syntax Rules](#3-function-syntax-rules)
4.  [Function Registration Rules](#4-function-registration-rules)
5.  [Validator Rules](#5-validator-rules)
6.  [Schema Rules](#6-schema-rules)
7.  [Query Rules](#7-query-rules)
8.  [Mutation Rules](#8-mutation-rules)
9.  [Action Rules](#9-action-rules)
10. [Function Calling Rules](#10-function-calling-rules)
11. [Function Reference Rules](#11-function-reference-rules)
12. [HTTP Endpoint Rules](#12-http-endpoint-rules)
13. [Pagination Rules](#13-pagination-rules)
14. [Scheduling & Cron Rules](#14-scheduling--cron-rules)
15. [File Storage Rules](#15-file-storage-rules)
16. [Full-Text Search Rules](#16-full-text-search-rules)
17. [Vector Search Rules](#17-vector-search-rules)
18. [Authentication & Security Rules](#18-authentication--security-rules)
19. [Error Handling Rules](#19-error-handling-rules)
20. [TypeScript Rules](#20-typescript-rules)
21. [API Design Rules](#21-api-design-rules)
22. [Performance Rules](#22-performance-rules)
23. [ESLint & Tooling Rules](#23-eslint--tooling-rules)
24. [Environment & Configuration Rules](#24-environment--configuration-rules)
25. [Code Documentation Rules](#25-code-documentation-rules)
26. [Real-World Examples](#26-real-world-examples)

---

## 1. Philosophy & Core Principles

### The Convex Mindset

Convex is a **"backend-as-a-function"** platform. Do NOT carry over habits from
traditional REST, GraphQL, or raw SQL/NoSQL workflows. Embrace these core principles:

> **Rule 1:** Every **read** in your app should happen via a `query` function.
> **Rule 2:** Every **write** should happen via a `mutation` function.
> **Rule 3:** **Actions** are only for side effects — calling external APIs,
>             sending emails, running AI models, etc. They are NOT transactions.
> **Rule 4:** Mutations and queries should work with **fewer than a few hundred
>             records** and aim to finish in **under 100ms**.
> **Rule 5:** Always `await` every promise. Never leave floating promises.

### Public vs Internal — The Golden Rule

```
PUBLIC  (query, mutation, action)         → Exposed to the Internet
INTERNAL (internalQuery, internalMutation, internalAction) → Private, server-only
```

- NEVER expose sensitive functions as public. If a function should only be
  called by other Convex functions — make it `internal`.
- NEVER call an `internalQuery/Mutation/Action` from the client.

---

## 2. Project File Structure

```
convex/
├── _generated/          # ⚠️ Auto-generated — DO NOT edit manually
│   ├── api.ts           # api and internal objects — import from here
│   ├── dataModel.ts     # Doc, Id types — import from here
│   └── server.ts        # query, mutation, action, etc. — import from here
│
├── schema.ts            # ✅ ALL table definitions go here — always required
├── http.ts              # ✅ ALL HTTP endpoints go here
├── crons.ts             # ✅ ALL cron jobs go here
│
├── auth/                # Feature: Authentication
│   ├── users.ts         # Public auth functions (createUser, getUser, etc.)
│   └── sessions.ts      # Internal session management
│
├── messages/            # Feature: Messages
│   ├── index.ts         # Public functions
│   └── helpers.ts       # Internal helper functions (not routed)
│
├── products/            # Feature: Products
│   ├── index.ts
│   ├── actions.ts       # Product-related actions (AI, external APIs)
│   └── helpers.ts
│
├── lib/                 # Shared internal helpers (NOT Convex functions)
│   ├── validators.ts    # Reusable shared validators
│   └── utils.ts         # Pure helper utilities
│
└── constants.ts         # Shared constants
```

### File Naming Rules

```
convex/schema.ts          ← always this exact name
convex/http.ts            ← always this exact name
convex/crons.ts           ← always this exact name
convex/auth/users.ts      ← camelCase feature files
convex/messages/index.ts  ← public functions for a feature
convex/lib/utils.ts       ← shared non-function helpers
```

---

## 3. Function Syntax Rules

### ✅ ALWAYS use the new named-export object syntax

```typescript
import { query, mutation, action } from "./_generated/server";
import { v } from "convex/values";

// ✅ Correct — new syntax with full arg and return validators
export const getUser = query({
  args: { userId: v.id("users") },
  returns: v.union(
    v.object({
      _id: v.id("users"),
      _creationTime: v.number(),
      name: v.string(),
      email: v.string(),
    }),
    v.null()
  ),
  handler: async (ctx, args) => {
    return await ctx.db.get(args.userId);
  },
});
```

### ❌ NEVER use the old default export syntax

```typescript
// ❌ Wrong — deprecated pattern
export default query(async (ctx, args) => {
  // ...
});
```

### ✅ ALWAYS include both `args` AND `returns` validators on every function

```typescript
// ✅ Correct — even if nothing is returned, use v.null()
export const deleteMessage = mutation({
  args: { messageId: v.id("messages") },
  returns: v.null(),                     // ← required even if returning null
  handler: async (ctx, args) => {
    await ctx.db.delete(args.messageId);
    return null;
  },
});
```

---

## 4. Function Registration Rules

### Registration Cheat Sheet

| Visibility | Query              | Mutation              | Action              |
|------------|--------------------|-----------------------|---------------------|
| **Public** | `query`            | `mutation`            | `action`            |
| **Private**| `internalQuery`    | `internalMutation`    | `internalAction`    |

### Rules

- Use `query` / `mutation` / `action` for **public API** functions only.
- Use `internalQuery` / `internalMutation` / `internalAction` for **server-only** logic.
- NEVER register a function through `api` or `internal` objects — those are
  for **calling** functions, not registering them.
- NEVER make a sensitive or admin-only function public. Always use `internal`
  variants for privileged operations.

```typescript
import {
  query,
  mutation,
  action,
  internalQuery,
  internalMutation,
  internalAction,
} from "./_generated/server";

// ✅ Public — safe for anyone to call
export const listProducts = query({ ... });

// ✅ Internal — server-only, not exposed to the internet
export const computeAnalytics = internalMutation({ ... });
```

---

## 5. Validator Rules

### Complete Convex Type & Validator Reference

| Convex Type | TS/JS Type    | Example              | Validator             | Notes                                           |
|-------------|---------------|----------------------|-----------------------|-------------------------------------------------|
| Id          | string        | `doc._id`            | `v.id("tableName")`   | Always use table-specific `v.id()`              |
| Null        | null          | `null`               | `v.null()`            | Use `null`, NOT `undefined`                     |
| Int64       | bigint        | `3n`                 | `v.int64()`           | Range: -2^63 to 2^63-1                          |
| Float64     | number        | `3.14`               | `v.number()`          | All IEEE-754 doubles supported                  |
| Boolean     | boolean       | `true`               | `v.boolean()`         |                                                 |
| String      | string        | `"hello"`            | `v.string()`          | Max ~1MB, valid Unicode only                    |
| Bytes       | ArrayBuffer   | `new ArrayBuffer(8)` | `v.bytes()`           | Max ~1MB                                        |
| Array       | Array         | `[1, "a"]`           | `v.array(values)`     | Max 8192 items                                  |
| Object      | Object        | `{a: "b"}`           | `v.object({...})`     | Max 1024 entries, no `$` or `_` field prefixes  |
| Record      | Record        | `{"a": "1"}`         | `v.record(k, v)`      | Keys: ASCII, nonempty, no `$` or `_` prefix     |

### New Validator Helpers (v1.29.0+)

```typescript
import { v } from "convex/values";

// ✅ v.nullable() — shorthand for v.union(v.string(), v.null())
const optionalName = v.nullable(v.string());

// ✅ validator.pick() — pick specific fields from an object validator
const userValidator = v.object({
  name: v.string(),
  email: v.string(),
  role: v.string(),
});
const publicUserValidator = userValidator.pick(["name", "email"]);

// ✅ validator.omit() — omit fields from an object validator
const noRoleValidator = userValidator.omit(["role"]);

// ✅ validator.extend() — add fields to an object validator
const adminUserValidator = userValidator.extend({
  adminLevel: v.number(),
});

// ✅ validator.partial() — make all fields optional
const partialUserValidator = userValidator.partial();
```

### Array Validator Example

```typescript
export const example = mutation({
  args: {
    tags: v.array(v.string()),
    scores: v.array(v.union(v.string(), v.number())),
  },
  returns: v.null(),
  handler: async (ctx, args) => {
    return null;
  },
});
```

### Discriminated Union Validator (Schema Level)

```typescript
import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  tasks: defineTable(
    v.union(
      v.object({
        kind: v.literal("pending"),
        createdAt: v.number(),
      }),
      v.object({
        kind: v.literal("completed"),
        completedAt: v.number(),
        result: v.string(),
      }),
      v.object({
        kind: v.literal("failed"),
        error: v.string(),
      })
    )
  ),
});
```

### Deprecation Rules

```typescript
// ❌ Deprecated — do NOT use
v.bigint()   // use v.int64() instead
v.map()      // not supported — use v.record() instead
v.set()      // not supported — use v.array() instead
```

---

## 6. Schema Rules

### ✅ Always define your schema in `convex/schema.ts`

```typescript
import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  users: defineTable({
    name: v.string(),
    email: v.string(),
    role: v.union(
      v.literal("admin"),
      v.literal("user"),
      v.literal("moderator")
    ),
    clerkId: v.string(),
    avatarUrl: v.optional(v.string()),
  })
    .index("by_clerk_id", ["clerkId"])
    .index("by_email", ["email"])
    .index("by_role", ["role"]),

  messages: defineTable({
    channelId: v.id("channels"),
    authorId: v.id("users"),
    content: v.string(),
    isDeleted: v.optional(v.boolean()),
  })
    .index("by_channel", ["channelId"])
    .index("by_channel_and_author", ["channelId", "authorId"])
    .searchIndex("search_content", {
      searchField: "content",
      filterFields: ["channelId"],
    }),

  channels: defineTable({
    name: v.string(),
    description: v.optional(v.string()),
    isPrivate: v.boolean(),
  }).index("by_name", ["name"]),
});
```

### Schema Rules

- **Always** define your schema in `convex/schema.ts` — never skip this.
- **Always** add indexes for any field you will query by (especially foreign keys
  like `userId`, `channelId`, `teamId`). Add basic indexes from the start,
  not after performance issues emerge.
- **Always** name indexes with `by_` prefix followed by all indexed fields:
  - Single field: `by_user_id`
  - Multiple fields: `by_channel_and_author`
- Index fields **must be queried in the same order** they are defined.
  Create separate indexes if you need different ordering.
- Use `v.optional()` for nullable fields — do NOT store `undefined`.
- **Keep documents flat.** Do not deeply nest arrays of objects inside
  a document. Use relationships via IDs instead.
- System fields `_id` (`v.id(tableName)`) and `_creationTime` (`v.number()`)
  are automatically added — do NOT redefine them.
- Check the generated `_generated/` folder into your Git repo so the project
  typechecks without needing to run `npx convex dev` first.

```typescript
// ✅ Good — flat document with a relationship
// users table: { name, email }
// posts table: { authorId: v.id("users"), content, title }

// ❌ Bad — deeply nested data
// users table: { name, posts: [{ content, comments: [...] }] }
```

---

## 7. Query Rules

### Core Query Rules

- **NEVER** use `ctx.db.query("table")` without a `.withIndex()` in production.
  A bare query performs a **full table scan** which will bottleneck at scale.
- **NEVER** use `.filter()` as a replacement for an index. Use `.withIndex()`
  for all indexed lookups.
- Filtering in code (after `.collect()`) is acceptable and has the same
  performance as `.filter()`, but `.withIndex()` conditions are always faster.
- Use `.unique()` when you expect exactly one result. It throws if multiple
  documents match.
- Use `.first()` when you want the first match and are okay with `null`.

```typescript
import { query } from "./_generated/server";
import { v } from "convex/values";

// ✅ Good — uses withIndex for efficient lookup
export const getMessagesByChannel = query({
  args: { channelId: v.id("channels") },
  returns: v.array(
    v.object({
      _id: v.id("messages"),
      _creationTime: v.number(),
      channelId: v.id("channels"),
      authorId: v.id("users"),
      content: v.string(),
    })
  ),
  handler: async (ctx, args) => {
    return await ctx.db
      .query("messages")
      .withIndex("by_channel", (q) => q.eq("channelId", args.channelId))
      .order("desc")
      .collect();
  },
});

// ❌ Bad — full table scan
export const badQuery = query({
  args: { channelId: v.id("channels") },
  returns: v.array(v.any()),
  handler: async (ctx, args) => {
    return await ctx.db
      .query("messages")
      .filter((q) => q.eq(q.field("channelId"), args.channelId)) // ← NO INDEX!
      .collect();
  },
});
```

### Ordering Rules

- Default order is **ascending** by `_creationTime`.
- Use `.order("asc")` or `.order("desc")` explicitly for clarity.
- Index-based queries are ordered by the index fields — no slow table scans.

### Async Iteration Rule

```typescript
// ✅ Good — async iteration, no .collect()
for await (const doc of ctx.db.query("messages").withIndex("by_channel", q =>
  q.eq("channelId", args.channelId)
)) {
  // process each document
}

// ❌ Avoid — don't mix .collect()/.take() with async iteration
```

### Deletion via Query

```typescript
// ✅ Correct — Convex does NOT support .delete() on a query
const messages = await ctx.db
  .query("messages")
  .withIndex("by_channel", (q) => q.eq("channelId", args.channelId))
  .collect();

for (const message of messages) {
  await ctx.db.delete(message._id);
}
```

---

## 8. Mutation Rules

### Core Mutation Rules

- `ctx.db.insert(table, data)` — creates a new document, returns its `_id`.
- `ctx.db.patch(id, fields)` — **shallow merge** update. Throws if doc not found.
- `ctx.db.replace(id, data)` — **full replacement** of a document. Throws if not found.
- `ctx.db.delete(id)` — deletes a document by `_id`.
- Starting from `convex@1.31.0`, `db.get`, `db.patch`, `db.replace`, and
  `db.delete` take the table name as an additional parameter for type safety.

```typescript
import { mutation } from "./_generated/server";
import { v } from "convex/values";

export const updateUserProfile = mutation({
  args: {
    userId: v.id("users"),
    name: v.optional(v.string()),
    avatarUrl: v.optional(v.string()),
  },
  returns: v.null(),
  handler: async (ctx, args) => {
    const { userId, ...updates } = args;

    // ✅ patch — only updates specified fields
    await ctx.db.patch(userId, updates);

    // ✅ replace — replaces the ENTIRE document (include all required fields)
    // await ctx.db.replace(userId, { name: "New Name", email: "...", role: "user", clerkId: "..." });

    return null;
  },
});
```

### Mutation as a Transaction

- Every mutation is a **fully serializable transaction**.
- If a mutation throws, ALL writes inside it are rolled back automatically.
- Use this to your advantage — keep writes together in one mutation rather
  than splitting across multiple mutations, which introduces race conditions.

```typescript
// ✅ Good — all writes in one atomic mutation
export const createPostWithNotification = mutation({
  args: { authorId: v.id("users"), content: v.string() },
  returns: v.id("posts"),
  handler: async (ctx, args) => {
    const postId = await ctx.db.insert("posts", {
      authorId: args.authorId,
      content: args.content,
    });
    await ctx.db.insert("notifications", {
      userId: args.authorId,
      type: "post_created",
      postId,
    });
    return postId;
  },
});
```

---

## 9. Action Rules

### Core Action Rules

- Actions are for **side effects only**: external API calls, AI models, emails,
  file processing, payments, etc.
- Actions do **NOT** have access to `ctx.db`. Never try to use it inside an action.
- Actions are **NOT transactions** — they don't roll back on failure.
- Instead, trigger actions from a **mutation** that records intent first, then
  schedules the action. This enables progress tracking and retry logic.
- Actions can run in two runtimes: **V8** (default, edge-like) and **Node.js**.
- Add `"use node";` at the top of any file that uses Node.js built-in modules.

```typescript
"use node"; // ← Required when using Node.js modules (crypto, fs, path, etc.)

import { internalAction } from "./_generated/server";
import { v } from "convex/values";
import { internal } from "./_generated/api";
import OpenAI from "openai";

/*
 * Action Name:  generateAiSummary
 * Description:  Calls OpenAI to summarize a document. Writes result back
 *               via an internal mutation after completion.
 * Args:         documentId (Id<"documents">) — document to summarize
 * Returns:      null
 */
export const generateAiSummary = internalAction({
  args: { documentId: v.id("documents") },
  returns: v.null(),
  handler: async (ctx, args) => {
    // ✅ Load data via internal query — not ctx.db
    const document = await ctx.runQuery(
      internal.documents.getDocumentForSummary,
      { documentId: args.documentId }
    );

    if (!document) throw new Error("Document not found");

    const openai = new OpenAI();
    const completion = await openai.chat.completions.create({
      model: "gpt-4o",
      messages: [
        {
          role: "user",
          content: `Summarize this document: ${document.content}`,
        },
      ],
    });

    const summary = completion.choices[0].message.content;
    if (!summary) throw new Error("No summary generated");

    // ✅ Persist result via internal mutation — not ctx.db
    await ctx.runMutation(internal.documents.saveSummary, {
      documentId: args.documentId,
      summary,
    });

    return null;
  },
});
```

### Action Anti-Patterns

```typescript
// ❌ Bad — calling action directly from client for a mutation-like operation
// Trigger a mutation instead, which schedules the action

// ❌ Bad — using ctx.db inside an action
handler: async (ctx, args) => {
  const user = await ctx.db.get(args.userId); // ← RUNTIME ERROR
}

// ❌ Bad — splitting data writes across multiple runMutation calls
// unnecessarily — race conditions can occur
```

### ✅ Recommended Pattern: Mutation → Schedules Action

```typescript
// ✅ Trigger actions from mutations — not directly from the browser
export const requestDocumentSummary = mutation({
  args: { documentId: v.id("documents") },
  returns: v.null(),
  handler: async (ctx, args) => {
    // Record intent first
    await ctx.db.patch(args.documentId, { summaryStatus: "pending" });
    // Schedule the action
    await ctx.scheduler.runAfter(
      0,
      internal.documents.generateAiSummary,
      { documentId: args.documentId }
    );
    return null;
  },
});
```

---

## 10. Function Calling Rules

### Call Methods by Context

| You are in...      | You can call...                                      |
|--------------------|------------------------------------------------------|
| `query`            | `ctx.runQuery`                                       |
| `mutation`         | `ctx.runQuery`, `ctx.runMutation`                    |
| `action`           | `ctx.runQuery`, `ctx.runMutation`, `ctx.runAction`   |

### Cross-Runtime Actions

- Only use `ctx.runAction` to call another action if you **must** cross
  runtimes (e.g. V8 → Node.js).
- If you're in the **same runtime**, extract the shared logic into a plain
  async helper function and call it directly — this avoids unnecessary
  round-trips.

```typescript
// ✅ Good — shared logic extracted to a helper, called directly
async function sharedHelper(data: string): Promise<string> {
  return data.trim().toLowerCase();
}

export const actionA = internalAction({
  args: { text: v.string() },
  returns: v.null(),
  handler: async (ctx, args) => {
    const result = await sharedHelper(args.text); // ← direct call, no overhead
    return null;
  },
});
```

### Type Annotation for Same-File Calls

```typescript
// ✅ Add type annotation to work around TypeScript circularity
export const g = query({
  args: {},
  returns: v.null(),
  handler: async (ctx) => {
    const result: string = await ctx.runQuery(api.example.f, { name: "Yasir" });
    return null;
  },
});
```

### Always Pass a FunctionReference

```typescript
// ✅ Good — passing a FunctionReference
await ctx.runQuery(api.users.getUser, { userId: args.userId });
await ctx.runMutation(internal.messages.writeMessage, { content: "Hello" });

// ❌ Bad — passing the function directly
await ctx.runQuery(getUser, { userId: args.userId }); // ← does NOT work
```

---

## 11. Function Reference Rules

```typescript
// api — for PUBLIC functions (query, mutation, action)
import { api } from "./_generated/api";
api.messages.index.listMessages      // convex/messages/index.ts → listMessages (public)

// internal — for PRIVATE functions (internalQuery, internalMutation, internalAction)
import { internal } from "./_generated/api";
internal.messages.index.processQueue // convex/messages/index.ts → processQueue (internal)

// File-based routing examples:
// convex/users.ts → exported function `getUser` = api.users.getUser
// convex/auth/sessions.ts → exported function `validateSession` = internal.auth.sessions.validateSession
// convex/messages/index.ts → exported function `sendMessage` = api.messages.index.sendMessage
```

---

## 12. HTTP Endpoint Rules

- ALL HTTP endpoints must be defined in `convex/http.ts`.
- HTTP endpoints require `httpAction` and a `httpRouter`.
- The registered path is **exact** — `/api/webhook` maps to `/api/webhook`.
- Always export the router as `default`.

```typescript
import { httpRouter } from "convex/server";
import { httpAction } from "./_generated/server";
import { internal } from "./_generated/api";

const http = httpRouter();

/*
 * Endpoint:    POST /api/webhooks/stripe
 * Description: Handles incoming Stripe webhook events for payment processing.
 */
http.route({
  path: "/api/webhooks/stripe",
  method: "POST",
  handler: httpAction(async (ctx, req) => {
    const body = await req.text();
    const signature = req.headers.get("stripe-signature");

    if (!signature) {
      return new Response("Missing signature", { status: 400 });
    }

    await ctx.runMutation(internal.payments.processStripeWebhook, {
      body,
      signature,
    });

    return new Response("OK", { status: 200 });
  }),
});

/*
 * Endpoint:    GET /api/health
 * Description: Health check endpoint for uptime monitoring.
 */
http.route({
  path: "/api/health",
  method: "GET",
  handler: httpAction(async () => {
    return new Response(JSON.stringify({ status: "ok" }), {
      status: 200,
      headers: { "Content-Type": "application/json" },
    });
  }),
});

export default http;
```

---

## 13. Pagination Rules

- Use `paginationOptsValidator` from `convex/server` for paginated queries.
- Always pass `paginationOpts` as an arg when using `.paginate()`.
- A `.paginate()` result returns: `{ page, isDone, continueCursor }`.
- Use `paginationResultValidator()` (v1.29.0+) as your `returns` validator
  for paginated queries.

```typescript
import { v } from "convex/values";
import { query } from "./_generated/server";
import { paginationOptsValidator, paginationResultValidator } from "convex/server";

/*
 * Function Name: listMessagesPaginated
 * Description:   Returns paginated messages for a given channel,
 *                ordered by most recent first.
 * Args:          channelId (Id<"channels">), paginationOpts (PaginationOpts)
 * Returns:       PaginationResult<Doc<"messages">>
 */
export const listMessagesPaginated = query({
  args: {
    channelId: v.id("channels"),
    paginationOpts: paginationOptsValidator,
  },
  returns: paginationResultValidator(
    v.object({
      _id: v.id("messages"),
      _creationTime: v.number(),
      channelId: v.id("channels"),
      authorId: v.id("users"),
      content: v.string(),
    })
  ),
  handler: async (ctx, args) => {
    return await ctx.db
      .query("messages")
      .withIndex("by_channel", (q) => q.eq("channelId", args.channelId))
      .order("desc")
      .paginate(args.paginationOpts);
  },
});
```

---

## 14. Scheduling & Cron Rules

### Scheduler Rules

- Always `await` scheduler calls — never leave them floating.
- `ctx.scheduler.runAfter(delayMs, functionRef, args)` — delay in milliseconds.
- `ctx.scheduler.runAt(timestamp, functionRef, args)` — schedule at exact time.
- Prefer scheduling actions from mutations (not from other actions) when possible.

```typescript
// ✅ Good — awaited scheduler call inside a mutation
await ctx.scheduler.runAfter(
  0,
  internal.emails.sendWelcomeEmail,
  { userId: newUserId }
);

// ✅ Schedule 24 hours later
await ctx.scheduler.runAfter(
  24 * 60 * 60 * 1000,
  internal.reminders.sendReminder,
  { userId: args.userId }
);
```

### Cron Rules

- Only use `crons.interval()` or `crons.cron()` — NEVER use the deprecated
  `crons.hourly()`, `crons.daily()`, or `crons.weekly()` helpers.
- Both methods take a `FunctionReference` — NEVER pass the function directly.
- Define all crons in `convex/crons.ts` and export as `default`.
- If a cron calls an internal function from the same file, STILL import
  `internal` from `_generated/api`.

```typescript
// convex/crons.ts
import { cronJobs } from "convex/server";
import { internal } from "./_generated/api";
import { internalMutation, internalAction } from "./_generated/server";
import { v } from "convex/values";

/*
 * Function Name: cleanupExpiredSessions
 * Description:   Deletes sessions that have been expired for more than 30 days.
 * Args:          none
 * Returns:       null
 */
export const cleanupExpiredSessions = internalMutation({
  args: {},
  returns: v.null(),
  handler: async (ctx) => {
    const thirtyDaysAgo = Date.now() - 30 * 24 * 60 * 60 * 1000;
    const expired = await ctx.db
      .query("sessions")
      .withIndex("by_expiry", (q) => q.lt("expiresAt", thirtyDaysAgo))
      .collect();

    for (const session of expired) {
      await ctx.db.delete(session._id);
    }
    return null;
  },
});

/*
 * Function Name: generateDailyReport
 * Description:   Generates and emails the daily analytics report.
 * Args:          none
 * Returns:       null
 */
export const generateDailyReport = internalAction({
  args: {},
  returns: v.null(),
  handler: async (ctx) => {
    // ... generate report logic
    return null;
  },
});

const crons = cronJobs();

// ✅ crons.interval — run every 2 hours
crons.interval(
  "cleanup expired sessions",
  { hours: 2 },
  internal.crons.cleanupExpiredSessions,
  {}
);

// ✅ crons.cron — run at specific time (daily at 8:00 AM UTC)
crons.cron(
  "daily analytics report",
  "0 8 * * *",
  internal.crons.generateDailyReport,
  {}
);

export default crons;
```

---

## 15. File Storage Rules

- Convex stores files as `Blob` objects. Convert to/from `Blob` when uploading
  or downloading.
- `ctx.storage.getUrl(storageId)` — returns a signed URL string, or `null`
  if file doesn't exist.
- `ctx.storage.generateUploadUrl()` — generates a URL the client uses to
  upload a file directly. Call from a `mutation`.
- NEVER use deprecated `ctx.storage.getMetadata()`.
- To get file metadata, query the `_storage` system table using
  `ctx.db.system.get(fileId)`.

```typescript
import { mutation, query } from "./_generated/server";
import { v } from "convex/values";
import { Id } from "./_generated/dataModel";

// System storage document type
type FileMetadata = {
  _id: Id<"_storage">;
  _creationTime: number;
  contentType?: string;
  sha256: string;
  size: number;
};

/*
 * Function Name: generateUploadUrl
 * Description:   Generates a secure upload URL for direct client file upload.
 * Args:          none
 * Returns:       string — the upload URL
 */
export const generateUploadUrl = mutation({
  args: {},
  returns: v.string(),
  handler: async (ctx) => {
    // ✅ Verify user is authenticated before generating URL
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) throw new Error("Not authenticated");
    return await ctx.storage.generateUploadUrl();
  },
});

/*
 * Function Name: getFileMetadata
 * Description:   Retrieves metadata for a stored file using the system table.
 * Args:          fileId (Id<"_storage">)
 * Returns:       FileMetadata | null
 */
export const getFileMetadata = query({
  args: { fileId: v.id("_storage") },
  returns: v.null(),
  handler: async (ctx, args) => {
    // ✅ Use ctx.db.system.get — NOT ctx.storage.getMetadata (deprecated)
    const metadata: FileMetadata | null = await ctx.db.system.get(args.fileId);
    console.log(metadata);
    return null;
  },
});

/*
 * Function Name: saveUploadedFile
 * Description:   Saves a reference to an uploaded file in the documents table.
 * Args:          storageId (Id<"_storage">), name (string)
 * Returns:       Id<"documents">
 */
export const saveUploadedFile = mutation({
  args: {
    storageId: v.id("_storage"),
    name: v.string(),
  },
  returns: v.id("documents"),
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) throw new Error("Not authenticated");

    return await ctx.db.insert("documents", {
      storageId: args.storageId,
      name: args.name,
      uploadedBy: identity.subject,
    });
  },
});
```

---

## 16. Full-Text Search Rules

- Define a `searchIndex` in your schema on the table you want to search.
- Use `.withSearchIndex()` in your query — not `.filter()`.
- Always use `.take(n)` after a search — paginate does NOT work with search.

```typescript
// convex/schema.ts — add search index
messages: defineTable({
  channelId: v.id("channels"),
  authorId: v.id("users"),
  content: v.string(),
})
  .index("by_channel", ["channelId"])
  .searchIndex("search_content", {
    searchField: "content",       // field to search
    filterFields: ["channelId"],  // fields available to filter on
  }),
```

```typescript
// ✅ Full-text search query
export const searchMessages = query({
  args: {
    channelId: v.id("channels"),
    searchText: v.string(),
  },
  returns: v.array(
    v.object({
      _id: v.id("messages"),
      _creationTime: v.number(),
      channelId: v.id("channels"),
      authorId: v.id("users"),
      content: v.string(),
    })
  ),
  handler: async (ctx, args) => {
    return await ctx.db
      .query("messages")
      .withSearchIndex("search_content", (q) =>
        q.search("content", args.searchText).eq("channelId", args.channelId)
      )
      .take(20); // ← always use .take() with search, not .collect()
  },
});
```

---

## 17. Vector Search Rules

- Define a `vectorIndex` in your schema for AI-powered semantic search.
- Generate embeddings in an **action** (not query/mutation) since it requires
  an external API call.
- Store embeddings as `v.array(v.float64())` or `v.array(v.number())`.

```typescript
// convex/schema.ts
documents: defineTable({
  content: v.string(),
  embedding: v.optional(v.array(v.number())),
  authorId: v.id("users"),
}).vectorIndex("by_embedding", {
  vectorField: "embedding",
  dimensions: 1536,             // match your embedding model's dimensions
  filterFields: ["authorId"],
}),
```

```typescript
// ✅ Vector search action
export const semanticSearch = internalAction({
  args: { query: v.string(), authorId: v.id("users") },
  returns: v.null(),
  handler: async (ctx, args) => {
    const openai = new OpenAI();

    // Generate embedding for the search query
    const embeddingResult = await openai.embeddings.create({
      model: "text-embedding-3-small",
      input: args.query,
    });
    const embedding = embeddingResult.data[0].embedding;

    // Run vector search via internal query
    const results = await ctx.vectorSearch("documents", "by_embedding", {
      vector: embedding,
      limit: 10,
      filter: (q) => q.eq("authorId", args.authorId),
    });

    return null;
  },
});
```

---

## 18. Authentication & Security Rules

### Core Security Rules

- **ALWAYS** verify authentication in **every** sensitive query, mutation, and action.
- NEVER rely solely on frontend UI guards (hidden buttons, conditional renders)
  to protect data. Enforce access in the Convex function itself.
- Use `ctx.auth.getUserIdentity()` to get the authenticated user's identity.
  Returns `null` if the user is not authenticated.

```typescript
import { mutation, query } from "./_generated/server";
import { v } from "convex/values";

/*
 * Function Name: getMyProfile
 * Description:   Fetches the currently authenticated user's profile.
 * Args:          none
 * Returns:       User object or null
 */
export const getMyProfile = query({
  args: {},
  returns: v.union(
    v.object({
      _id: v.id("users"),
      _creationTime: v.number(),
      name: v.string(),
      email: v.string(),
    }),
    v.null()
  ),
  handler: async (ctx) => {
    // ✅ Always check auth first
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) return null;

    return await ctx.db
      .query("users")
      .withIndex("by_clerk_id", (q) => q.eq("clerkId", identity.subject))
      .unique();
  },
});

/*
 * Function Name: deletePost
 * Description:   Deletes a post. Only the post's author can delete it.
 * Args:          postId (Id<"posts">)
 * Returns:       null
 */
export const deletePost = mutation({
  args: { postId: v.id("posts") },
  returns: v.null(),
  handler: async (ctx, args) => {
    // ✅ Check authentication
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) throw new Error("Not authenticated");

    const post = await ctx.db.get(args.postId);
    if (!post) throw new Error("Post not found");

    // ✅ Check authorization — only owner can delete
    if (post.authorClerkId !== identity.subject) {
      throw new Error("Not authorized");
    }

    await ctx.db.delete(args.postId);
    return null;
  },
});
```

### Security Checklist

```
✅ ctx.auth.getUserIdentity() called at top of every protected function
✅ Authorization checks (ownership/role) after authentication
✅ Sensitive operations use internalMutation/internalAction — not public
✅ HTTP webhook endpoints verify signatures before processing
✅ Never expose internal IDs or sensitive data in public query returns
✅ Use v.id("tableName") for document IDs — not v.string()
```

---

## 19. Error Handling Rules

- Throw JavaScript `Error` objects with descriptive messages.
- Errors thrown inside queries/mutations cause the entire operation to roll back.
- In actions, errors do NOT auto-retry unless scheduled with retry logic.
- Use try/catch in actions when calling external APIs, and handle partial
  failures gracefully by writing status back via mutation.

```typescript
// ✅ Good — descriptive errors with context
handler: async (ctx, args) => {
  const user = await ctx.db.get(args.userId);
  if (!user) {
    throw new Error(`User not found: ${args.userId}`);
  }

  const channel = await ctx.db.get(args.channelId);
  if (!channel) {
    throw new Error(`Channel not found: ${args.channelId}`);
  }

  if (!user.isMember) {
    throw new Error(`User ${args.userId} is not a member of this channel`);
  }
  // ...
};

// ✅ Good — graceful action failure with status tracking
export const processPayment = internalAction({
  args: { orderId: v.id("orders") },
  returns: v.null(),
  handler: async (ctx, args) => {
    try {
      // external payment call
      await ctx.runMutation(internal.orders.markPaymentSuccess, {
        orderId: args.orderId,
      });
    } catch (error) {
      await ctx.runMutation(internal.orders.markPaymentFailed, {
        orderId: args.orderId,
        error: error instanceof Error ? error.message : "Unknown error",
      });
    }
    return null;
  },
});
```

---

## 20. TypeScript Rules

### ID Type Rules

```typescript
import { Doc, Id } from "./_generated/dataModel";

// ✅ Always use typed Ids — not plain string
function getUser(userId: Id<"users">) {}

// ❌ Bad — too loose
function getUser(userId: string) {}

// ✅ Use Doc<"tableName"> for full document types
function displayUser(user: Doc<"users">) {}
```

### Record Type Rules

```typescript
import { Id } from "./_generated/dataModel";

// ✅ Correct — fully typed Record
const userMap: Record<Id<"users">, string> = {};

// ✅ Correct — explicitly typed arrays
const userIds: Array<Id<"users">> = [];
```

### Discriminated Union Rules

```typescript
// ✅ Use as const for string literals in discriminated unions
type TaskStatus =
  | { kind: "pending" as const; createdAt: number }
  | { kind: "completed" as const; completedAt: number }
  | { kind: "failed" as const; error: string };
```

### Node.js Module Rules

```typescript
// ✅ Always add @types/node when using Node.js built-ins
// package.json:
// "devDependencies": { "@types/node": "^20.0.0" }

"use node"; // at top of file
import { createHash } from "crypto"; // ← Node.js built-in
```

### Generated Types in Git

- Always commit the `convex/_generated/` folder to your Git repo.
- This enables TypeScript to work without running `npx convex dev` first.

---

## 21. API Design Rules

### File-Based Routing

```
convex/
├── users.ts             → api.users.*
├── messages/
│   └── index.ts         → api.messages.index.*
├── products/
│   ├── index.ts         → api.products.index.*
│   └── reviews.ts       → api.products.reviews.*
└── internal/
    └── analytics.ts     → internal.internal.analytics.*
```

### Design Principles

- Group related functions together in the same file.
- One file per **feature** or **domain** in most cases.
- If a file grows beyond ~150 lines, split it into multiple files within
  a feature folder.
- Prefix all internal file names clearly: `actions.ts`, `helpers.ts`,
  `validators.ts`, etc.
- Avoid deeply nested folder structures — max 2 levels inside `convex/`.

---

## 22. Performance Rules

### Indexing Rules

```typescript
// ✅ Add indexes for any field you query by — especially foreign keys
users: defineTable({...})
  .index("by_clerk_id", ["clerkId"])   // ← always index auth foreign key
  .index("by_email", ["email"]),

posts: defineTable({...})
  .index("by_author", ["authorId"])    // ← foreign key index from the start
  .index("by_author_and_status", ["authorId", "status"]), // compound index
```

### Query Limits

- Avoid `.collect()` on large tables without a `.withIndex()` filter first.
- Use `.take(n)` when you only need a fixed number of results.
- Use `.paginate()` for large result sets in UI lists.
- Queries and mutations should aim to complete in **under 100ms**.
- Avoid running actions that process thousands of records in one call;
  batch the work and track progress with mutations.

### Avoid Floating Promises

```typescript
// ✅ Good — await all promises
await ctx.scheduler.runAfter(0, internal.emails.send, { userId });
await ctx.db.patch(userId, { lastSeen: Date.now() });

// ❌ Bad — floating promises cause unpredictable behavior
ctx.scheduler.runAfter(0, internal.emails.send, { userId }); // ← NO await!
```

---

## 23. ESLint & Tooling Rules

### ESLint Setup

```bash
# Install Convex ESLint plugin (out of beta as of v1.29.0)
npm install --save-dev eslint-plugin-convex
```

```js
// eslint.config.mjs
import convex from "eslint-plugin-convex";

export default [
  {
    plugins: { convex },
    rules: {
      ...convex.configs.recommended.rules,
      "@typescript-eslint/no-floating-promises": "error", // ← enforce await
    },
  },
];
```

### Recommended Rules

- `eslint-plugin-convex` — enforces Convex best practices automatically.
- `@typescript-eslint/no-floating-promises` — ensures all promises are awaited.
- `@typescript-eslint/strict-boolean-expressions` — avoids falsy type issues.

---

## 24. Environment & Configuration Rules

### Environment Variable Rules

```bash
# .env.local — never commit this file

# ✅ Convex deployment URL (auto-set by Convex CLI)
CONVEX_DEPLOYMENT=dev:your-deployment-name

# ✅ Public-facing URL (for Next.js or Vite client)
NEXT_PUBLIC_CONVEX_URL=https://your-deployment.convex.cloud

# ✅ External services (server-only, never exposed to client)
OPENAI_API_KEY=sk-...
STRIPE_SECRET_KEY=sk_live_...
RESEND_API_KEY=re_...
CLERK_SECRET_KEY=sk_...
```

### Setting Convex Environment Variables

```bash
# Set via Convex CLI (not .env for server-side secrets)
npx convex env set OPENAI_API_KEY sk-your-key
npx convex env set STRIPE_SECRET_KEY sk_live_your-key

# Access in Convex functions via process.env
const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });
```

### Convex-Specific Config Files

```typescript
// convex/constants.ts — shared constants across Convex functions
export const MAX_MESSAGE_LENGTH = 2000;
export const MAX_FILE_SIZE_BYTES = 10 * 1024 * 1024; // 10MB
export const DEFAULT_PAGE_SIZE = 20;
export const AI_MODEL = "gpt-4o" as const;
```

---

## 25. Code Documentation Rules

### File Header — Required on every `convex/` file

```typescript
/*
 * File Name:    authService.ts
 * Description:  Handles all user authentication operations including
 *               user creation, profile fetching, and session validation.
 * Author:       Yasir Khan
 * Created Date: 2026-02-21
 * Last Modified: 2026-02-21
 */
```

### Function Documentation — Required on all exported functions

```typescript
/*
 * Function Name: getUserById
 * Description:   Fetches a user document from the database by their unique ID.
 * Args:
 *   - userId (Id<"users">) — the unique Convex document ID of the user
 * Returns:       Doc<"users"> | null — the user document, or null if not found
 */
export const getUserById = query({
  args: { userId: v.id("users") },
  returns: v.union(v.object({ ... }), v.null()),
  handler: async (ctx, args) => {
    return await ctx.db.get(args.userId);
  },
});
```

### Mutation/Action Documentation Example

```typescript
/*
 * Function Name: sendMessage
 * Description:   Inserts a new message into a channel and schedules an
 *                AI response via an internal action.
 * Args:
 *   - channelId  (Id<"channels">) — the target channel
 *   - authorId   (Id<"users">)    — the message author
 *   - content    (string)         — the message body (max 2000 chars)
 * Returns:       null
 * Side Effects:  Schedules internal.messages.generateAiReply action
 */
export const sendMessage = mutation({
  args: {
    channelId: v.id("channels"),
    authorId: v.id("users"),
    content: v.string(),
  },
  returns: v.null(),
  handler: async (ctx, args) => { ... },
});
```

### TODO & FIXME Comments

```typescript
// TODO [Yasir Khan, 2026-02-21]: Replace with RAG component once ready.
// FIXME [Yasir Khan, 2026-02-21]: Rate limiting not yet enforced here.
```

---

## 26. Real-World Examples

### Example: Full Auth + Messaging Backend

#### convex/schema.ts

```typescript
/*
 * File Name:    schema.ts
 * Description:  Defines all database tables and indexes for the application.
 * Author:       Yasir Khan
 * Created Date: 2026-02-21
 */

import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  users: defineTable({
    name: v.string(),
    email: v.string(),
    clerkId: v.string(),
    role: v.union(v.literal("admin"), v.literal("user")),
    avatarUrl: v.optional(v.string()),
  })
    .index("by_clerk_id", ["clerkId"])
    .index("by_email", ["email"]),

  channels: defineTable({
    name: v.string(),
    description: v.optional(v.string()),
    isPrivate: v.boolean(),
    createdBy: v.id("users"),
  }).index("by_name", ["name"]),

  messages: defineTable({
    channelId: v.id("channels"),
    authorId: v.optional(v.id("users")), // optional for AI messages
    content: v.string(),
    isAiGenerated: v.boolean(),
  })
    .index("by_channel", ["channelId"])
    .searchIndex("search_content", {
      searchField: "content",
      filterFields: ["channelId"],
    }),
});
```

#### convex/auth/users.ts

```typescript
/*
 * File Name:    users.ts
 * Description:  Public user management functions: creation and fetching.
 * Author:       Yasir Khan
 * Created Date: 2026-02-21
 */

import { mutation, query } from "../_generated/server";
import { v } from "convex/values";

/*
 * Function Name: createOrGetUser
 * Description:   Creates a new user if they don't exist, or returns
 *                the existing user based on their Clerk identity.
 * Args:          name (string), email (string)
 * Returns:       Id<"users">
 */
export const createOrGetUser = mutation({
  args: {
    name: v.string(),
    email: v.string(),
  },
  returns: v.id("users"),
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) throw new Error("Not authenticated");

    const existing = await ctx.db
      .query("users")
      .withIndex("by_clerk_id", (q) => q.eq("clerkId", identity.subject))
      .unique();

    if (existing) return existing._id;

    return await ctx.db.insert("users", {
      name: args.name,
      email: args.email,
      clerkId: identity.subject,
      role: "user",
    });
  },
});

/*
 * Function Name: getMyProfile
 * Description:   Returns the authenticated user's profile document.
 * Args:          none
 * Returns:       Doc<"users"> | null
 */
export const getMyProfile = query({
  args: {},
  returns: v.union(
    v.object({
      _id: v.id("users"),
      _creationTime: v.number(),
      name: v.string(),
      email: v.string(),
      clerkId: v.string(),
      role: v.union(v.literal("admin"), v.literal("user")),
      avatarUrl: v.optional(v.string()),
    }),
    v.null()
  ),
  handler: async (ctx) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) return null;

    return await ctx.db
      .query("users")
      .withIndex("by_clerk_id", (q) => q.eq("clerkId", identity.subject))
      .unique();
  },
});
```

#### convex/messages/index.ts

```typescript
/*
 * File Name:    index.ts (messages feature)
 * Description:  Public messaging functions: listing, sending, and searching.
 * Author:       Yasir Khan
 * Created Date: 2026-02-21
 */

import { mutation, query, internalMutation, internalAction } from "../_generated/server";
import { v } from "convex/values";
import { internal } from "../_generated/api";
import { paginationOptsValidator, paginationResultValidator } from "convex/server";

/*
 * Function Name: listMessages
 * Description:   Returns the 20 most recent messages for a channel,
 *                in descending order (newest first).
 * Args:
 *   - channelId (Id<"channels">) — the channel to fetch messages from
 * Returns:       Array of message objects
 */
export const listMessages = query({
  args: { channelId: v.id("channels") },
  returns: v.array(
    v.object({
      _id: v.id("messages"),
      _creationTime: v.number(),
      channelId: v.id("channels"),
      authorId: v.optional(v.id("users")),
      content: v.string(),
      isAiGenerated: v.boolean(),
    })
  ),
  handler: async (ctx, args) => {
    return await ctx.db
      .query("messages")
      .withIndex("by_channel", (q) => q.eq("channelId", args.channelId))
      .order("desc")
      .take(20);
  },
});

/*
 * Function Name: sendMessage
 * Description:   Validates auth, inserts a new message, and schedules
 *                an AI response.
 * Args:
 *   - channelId (Id<"channels">) — target channel
 *   - content   (string)         — message content
 * Returns:       null
 * Side Effects:  Schedules internal AI response generation
 */
export const sendMessage = mutation({
  args: {
    channelId: v.id("channels"),
    content: v.string(),
  },
  returns: v.null(),
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) throw new Error("Not authenticated");

    const user = await ctx.db
      .query("users")
      .withIndex("by_clerk_id", (q) => q.eq("clerkId", identity.subject))
      .unique();
    if (!user) throw new Error("User profile not found");

    const channel = await ctx.db.get(args.channelId);
    if (!channel) throw new Error("Channel not found");

    await ctx.db.insert("messages", {
      channelId: args.channelId,
      authorId: user._id,
      content: args.content,
      isAiGenerated: false,
    });

    // Schedule AI response — do not await action from mutation
    await ctx.scheduler.runAfter(
      0,
      internal.messages.index.generateAiReply,
      { channelId: args.channelId }
    );

    return null;
  },
});

/*
 * Function Name: generateAiReply
 * Description:   Internal action that generates an AI response for the
 *                latest messages in a channel using OpenAI GPT-4o.
 * Args:          channelId (Id<"channels">)
 * Returns:       null
 */
export const generateAiReply = internalAction({
  args: { channelId: v.id("channels") },
  returns: v.null(),
  handler: async (ctx, args) => {
    const { OpenAI } = await import("openai");
    const openai = new OpenAI();

    const messages: Array<{ role: "user" | "assistant"; content: string }> =
      await ctx.runQuery(internal.messages.index.loadContext, {
        channelId: args.channelId,
      });

    const response = await openai.chat.completions.create({
      model: "gpt-4o",
      messages,
    });

    const content = response.choices[0]?.message?.content;
    if (!content) throw new Error("Empty AI response");

    await ctx.runMutation(internal.messages.index.writeAiMessage, {
      channelId: args.channelId,
      content,
    });

    return null;
  },
});

/*
 * Function Name: loadContext
 * Description:   Loads the 10 most recent messages formatted for the
 *                OpenAI chat completions API.
 * Args:          channelId (Id<"channels">)
 * Returns:       Array of { role, content } objects
 */
export const loadContext = query({
  args: { channelId: v.id("channels") },
  returns: v.array(
    v.object({
      role: v.union(v.literal("user"), v.literal("assistant")),
      content: v.string(),
    })
  ),
  handler: async (ctx, args) => {
    const messages = await ctx.db
      .query("messages")
      .withIndex("by_channel", (q) => q.eq("channelId", args.channelId))
      .order("desc")
      .take(10);

    const result: Array<{ role: "user" | "assistant"; content: string }> = [];

    for (const message of messages) {
      if (message.authorId) {
        const user = await ctx.db.get(message.authorId);
        result.push({
          role: "user" as const,
          content: `${user?.name ?? "Unknown"}: ${message.content}`,
        });
      } else {
        result.push({ role: "assistant" as const, content: message.content });
      }
    }

    return result;
  },
});

/*
 * Function Name: writeAiMessage
 * Description:   Inserts an AI-generated message into the channel.
 * Args:
 *   - channelId (Id<"channels">) — target channel
 *   - content   (string)         — AI message content
 * Returns:       null
 */
export const writeAiMessage = internalMutation({
  args: {
    channelId: v.id("channels"),
    content: v.string(),
  },
  returns: v.null(),
  handler: async (ctx, args) => {
    await ctx.db.insert("messages", {
      channelId: args.channelId,
      content: args.content,
      isAiGenerated: true,
    });
    return null;
  },
});
```

---

## 📌 Quick Reference Cheat Sheet

| Rule                           | ✅ Do This                            | ❌ Not This                          |
|--------------------------------|---------------------------------------|--------------------------------------|
| Function syntax                | New named-export object syntax        | Old default export syntax            |
| Return validator               | Always include `returns: v.null()`    | Omit `returns` validator             |
| Public vs private              | Use `internal*` for private functions | Use `query/mutation` for everything  |
| Query indexing                 | `.withIndex("by_field", ...)`         | `.filter(q => q.eq(...))`            |
| Deleting via query             | `.collect()` then loop `db.delete()`  | `.delete()` (doesn't exist)          |
| Context in actions             | `ctx.runQuery / ctx.runMutation`      | `ctx.db` inside actions              |
| Triggering actions             | Mutation schedules action             | Call action directly from client     |
| Awaiting promises              | Always `await` every promise          | Leave floating promises              |
| Cron methods                   | `crons.interval()` / `crons.cron()`   | `crons.hourly()` / `crons.daily()`   |
| File metadata                  | `ctx.db.system.get(fileId)`           | `ctx.storage.getMetadata()` (deprecated) |
| ID types                       | `Id<"users">` / `v.id("users")`       | `string` for document IDs            |
| Int64 validator                | `v.int64()`                           | `v.bigint()` (deprecated)            |
| Nullable validators            | `v.nullable(v.string())`              | Manual `v.union(v.string(), v.null())`|
| Schema                         | Always define in `convex/schema.ts`   | Skip schema / use untyped docs       |
| Indexes                        | Add from the start for FK fields      | Wait until app is slow               |
| Document shape                 | Keep documents flat + use relations   | Deeply nest arrays of objects        |
| Authentication                 | Check `ctx.auth.getUserIdentity()`    | Trust frontend auth guards only      |
| Environment secrets            | `npx convex env set KEY value`        | Hardcode secrets in source files     |

