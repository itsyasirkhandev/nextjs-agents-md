# Enhanced Prompt for AI Agents — Maximum Coding Excellence

Use this prompt to generate an `agents.md` file that will guide AI agents to write optimal code following Next.js, Convex, and Optimization best practices.

---

# Analyze this codebase and generate a hierarchical agents.md structure

## Context & Principles

You are helping create a **hierarchical agents.md system** for a Next.js + Convex project. This system is critical for AI coding agents to work efficiently with minimal token usage while following strict coding standards.

### Technology Stack
- **Frontend:** Next.js 16 (App Router) with React 19
- **Backend:** Convex (backend-as-a-function)
- **Styling:** Tailwind CSS
- **Language:** TypeScript (strict mode)

---

## CRITICAL RULES — Must Be Followed

### 🔴 NON-NEGOTIABLE CONVIEX RULES

```
1. EVERY function MUST have both `args` AND `returns` validators
2. ALWAYS use named-export object syntax (NEVER default export)
3. NEVER use ctx.db.query() without .withIndex() in production
4. ALWAYS await every promise — NEVER leave floating promises
5. Actions CANNOT use ctx.db — use ctx.runQuery/runMutation instead
6. ALL schema extensions MUST use v.optional() for backward compatibility
7. Indexes must be named with `by_` prefix: by_user_id, by_channel_and_author
8. NEVER make sensitive functions public — use internalQuery/internalMutation/internalAction
```

### 🔴 NON-NEGOTIABLE NEXT.JS RULES

```
1. Component files → PascalCase.tsx | Non-component files → camelCase.ts
2. App Router reserved files → lowercase (page.tsx, layout.tsx, etc.)
3. Private folders inside app/ → underscore prefix (_components/, _hooks/, _lib/)
4. Route Groups → parentheses ((auth)/, (main)/)
5. "use client" directive ONLY when component needs useState/useEffect/browser APIs
6. Keep "use client" boundaries as LOW in the component tree as possible
7. Server Actions in *.actions.ts files — keep them THIN, business logic in services
8. ALWAYS use absolute imports with @/ path alias
```

### 🔴 NON-NEGOTIABLE OPTIMIZATION RULES (LEVER Framework)

```
L — Leverage existing patterns FIRST
E — Extend BEFORE creating anything new
V — Verify through reactivity and existing data flows
E — Eliminate every source of duplication
R — Reduce complexity at every decision point
```

**Before writing ANY code, answer these 4 questions:**
1. Does this functionality already exist somewhere in this codebase?
2. Can I extend something that already exists?
3. Can I adapt an existing pattern instead of creating a new one?
4. Is the new code I am about to write reusable, or will it only be used once?

---

## Your Process

### Phase 1: Repository Analysis (10-15 minutes)

First, analyze the codebase structure and provide:

1. **Repository type**: Monorepo, multi-package, or simple single project?
2. **Primary technology stack**: Confirm Next.js version, Convex, styling approach
3. **Major directories/packages** that should have their own agents.md:
   - `convex/` — Backend functions (queries, mutations, actions)
   - `src/app/` — Next.js App Router routes
   - `src/components/` — Global shared components
   - `src/hooks/` — Global custom hooks
   - `src/services/` — Business logic / API clients
   - `src/lib/` — Third-party library wrappers
   - `src/types/` — TypeScript types
   - Feature folders inside `src/app/(main)/`

4. **Build system**: npm/pnpm/yarn? Turborepo?
5. **Testing setup**: Jest, Vitest, Playwright? Where are tests?
6. **Key patterns to document**:
   - Convex function patterns (query, mutation, action)
   - Next.js Server/Client component split
   - Authentication flow
   - Database schema and indexes

Present this as a **structured map** before generating any agents.md files.

---

### Phase 2: Generate Root agents.md

Create a **lightweight root agents.md** (~150-200 lines max) that includes:

#### Required Sections:

**1. Project Snapshot** (3-5 lines)
```
- Repo type (monorepo/simple)
- Tech: Next.js 16 (App Router) + Convex + Tailwind
- Note: Sub-packages have their own agents.md files
```

**2. Root Setup Commands** (5-10 lines)
```bash
# Install
pnpm install

# Development
pnpm dev

# Build
pnpm build

# Type check
pnpm typecheck

# Convex development
pnpm convex dev
```

**3. Universal Conventions** (10-15 lines)

**Code Style:**
- TypeScript strict mode
- PascalCase for React components, camelCase for functions/variables
- SCREAMING_SNAKE_CASE for constants
- Boolean variables: is/has/can/should prefix (isLoading, hasError, canEdit)

**Import Order:**
1. Node built-ins
2. External packages
3. Internal absolute imports (@/)
4. Relative imports
5. Type imports (last)

**File Headers:**
Every file MUST start with:
```ts
/*
 * File Name:     fileName.ts
 * Description:   Brief description of what this file does.
 * Author:        [Author Name]
 * Created Date:  YYYY-MM-DD
 * Last Modified: YYYY-MM-DD
 */
```

**4. Convex Quick Reference** (15-20 lines)

```typescript
// ✅ ALWAYS use this syntax
export const getUser = query({
  args: { userId: v.id("users") },
  returns: v.union(v.object({ /* ... */ }), v.null()),
  handler: async (ctx, args) => {
    return await ctx.db
      .query("users")
      .withIndex("by_id", (q) => q.eq("_id", args.userId))
      .unique();
  },
});

// ❌ NEVER use this syntax
export default query(async (ctx, args) => { /* ... */ });
```

**Function Types:**
| Type | Use For | Visibility |
|------|---------|------------|
| `query` | Read operations | Public |
| `mutation` | Write operations | Public |
| `action` | Side effects (API calls, AI, emails) | Public |
| `internalQuery` | Server-only reads | Private |
| `internalMutation` | Server-only writes | Private |
| `internalAction` | Server-only side effects | Private |

**5. Next.js Quick Reference** (10-15 lines)

```
Server Components (default):
- No "use client" directive
- Can fetch data directly
- Cannot use useState, useEffect, browser APIs

Client Components ("use client"):
- Only when interactivity needed
- Keep boundaries LOW in component tree
- Cannot import Server Components
```

**6. LEVER Framework Reminder** (5-10 lines)
```
Before creating NEW code:
1. Check if similar functionality exists
2. Can you EXTEND an existing query/mutation/component?
3. Add optional fields to existing tables instead of new tables
4. Consolidate multiple queries into one rich query
5. Use computed properties instead of new queries
```

**7. JIT Index - Directory Map** (10-20 lines)
```
## JIT Index (what to open, not what to paste)

### Package Structure
- Convex Backend: `convex/` → [see convex/agents.md](convex/agents.md)
- App Router: `src/app/` → [see src/app/agents.md](src/app/agents.md)
- Components: `src/components/` → [see src/components/agents.md](src/components/agents.md)
- Hooks: `src/hooks/` → [see src/hooks/agents.md](src/hooks/agents.md)
- Types: `src/types/` → shared TypeScript types

### Quick Find Commands
- Convex queries: `rg -n "export const.*= query" convex/`
- Convex mutations: `rg -n "export const.*= mutation" convex/`
- React components: `rg -n "export function|export const.*=" src/components`
- Server Actions: `rg -n '"use server"' src/`
- Database schema: `rg -n "defineTable" convex/schema.ts`
```

**8. Anti-Patterns to Avoid** (10-15 lines)
```
❌ Creating new tables when optional fields on existing tables work
❌ Multiple similar queries instead of one rich query with computed fields
❌ Polling instead of using Convex reactivity (useQuery)
❌ Floating promises without await
❌ ctx.db.query() without .withIndex()
❌ Using ctx.db inside actions
❌ Business logic in Server Actions (should be in services)
❌ "use client" at the top of component trees (push it down)
❌ Relative imports instead of @/ alias
```

**9. Performance Targets** (5-10 lines)
```
| Operation | Target | Warning | Critical |
|-----------|--------|---------|----------|
| Query time | < 100ms | 100-300ms | > 300ms |
| Mutation time | < 50ms | 50-150ms | > 150ms |
| Documents per query | < 100 | 100-1000 | > 1000 |
| New files per feature | 0-2 | 3-4 | > 4 |
```

**10. Definition of Done** (3-5 lines)
```
- [ ] TypeScript compiles without errors
- [ ] All Convex functions have args AND returns validators
- [ ] All queries use .withIndex()
- [ ] All promises are awaited
- [ ] "use client" only where necessary
- [ ] Tests pass (if applicable)
```

---

### Phase 3: Generate Sub-Folder agents.md Files

For EACH major directory identified in Phase 1, create a **detailed agents.md**:

---

#### `convex/agents.md` — Backend Functions

**1. Package Identity**
```
Convex backend functions for this project.
All database operations, real-time subscriptions, and server-side logic.
```

**2. File Structure**
```
convex/
├── _generated/     # Auto-generated — DO NOT EDIT
├── schema.ts       # ALL table definitions — required
├── http.ts         # HTTP endpoints
├── crons.ts        # Cron jobs
├── auth/           # Authentication functions
├── [feature]/      # Feature-specific functions
│   ├── index.ts    # Public functions
│   └── helpers.ts  # Internal helpers
└── lib/            # Shared utilities
```

**3. Function Syntax (MANDATORY)**
```typescript
// ✅ CORRECT — Always use this pattern
import { query, mutation, action } from "./_generated/server";
import { v } from "convex/values";

export const getMessages = query({
  args: {
    channelId: v.id("channels"),
    limit: v.optional(v.number()),
  },
  returns: v.array(
    v.object({
      _id: v.id("messages"),
      _creationTime: v.number(),
      channelId: v.id("channels"),
      content: v.string(),
    })
  ),
  handler: async (ctx, args) => {
    return await ctx.db
      .query("messages")
      .withIndex("by_channel", (q) => q.eq("channelId", args.channelId))
      .order("desc")
      .take(args.limit ?? 20);
  },
});

// ❌ WRONG — Deprecated, DO NOT USE
export default query(async (ctx, args) => { /* ... */ });
```

**4. Schema Rules**
```typescript
// Always in convex/schema.ts
export default defineSchema({
  users: defineTable({
    name: v.string(),
    email: v.string(),
    role: v.union(v.literal("admin"), v.literal("user")),
    avatarUrl: v.optional(v.string()), // ← ALWAYS optional for new fields
  })
    .index("by_email", ["email"])
    .index("by_role", ["role"]),
});

// Index naming: by_[field1]_and_[field2]
// Query MUST use fields in index order
```

**5. Query Rules**
```typescript
// ✅ ALWAYS use withIndex
await ctx.db
  .query("messages")
  .withIndex("by_channel", (q) => q.eq("channelId", args.channelId))
  .collect();

// ❌ NEVER do full table scan
await ctx.db.query("messages").filter(...) // WRONG!

// ✅ Use .take(n) for bounded lists
.take(20)

// ✅ Use .paginate() for large lists
.paginate(args.paginationOpts)
```

**6. Mutation Rules**
```typescript
// ✅ All writes in ONE mutation (atomic transaction)
export const createPostWithNotification = mutation({
  args: { content: v.string() },
  returns: v.id("posts"),
  handler: async (ctx, args) => {
    const postId = await ctx.db.insert("posts", { content: args.content });
    await ctx.db.insert("notifications", { postId, type: "new_post" });
    return postId; // Both succeed or both roll back
  },
});

// ❌ NEVER split related writes across mutations
```

**7. Action Rules**
```typescript
"use node"; // Required for Node.js modules

export const sendEmail = internalAction({
  args: { userId: v.id("users"), subject: v.string() },
  returns: v.null(),
  handler: async (ctx, args) => {
    // ❌ NO ctx.db here — use runQuery/runMutation instead
    const user = await ctx.runQuery(internal.users.getById, { 
      userId: args.userId 
    });
    
    // Send email...
    
    return null;
  },
});

// ✅ Trigger from mutation
export const requestEmail = mutation({
  handler: async (ctx, args) => {
    await ctx.db.insert("emails", { status: "pending" });
    await ctx.scheduler.runAfter(0, internal.emails.sendEmail, args);
  },
});
```

**8. Common Gotchas**
```
- Floating promises: ALWAYS await ctx.db.*, ctx.scheduler.*, ctx.run*
- Missing validators: EVERY function needs args AND returns
- Wrong imports: Import from "./_generated/server", NOT direct paths
- Table name typos: v.id("users") must match schema table name exactly
- Index order: Must query indexed fields in the order they're defined
```

---

#### `src/app/agents.md` — Next.js App Router

**1. Package Identity**
```
Next.js 16 App Router routes and pages.
Server Components by default, Client Components only when necessary.
```

**2. Route Structure**
```
src/app/
├── layout.tsx          # Root layout
├── page.tsx            # Home page (/)
├── loading.tsx         # Loading skeleton
├── error.tsx           # Error boundary
├── globals.css         # Global styles
│
├── (auth)/             # Route Group: Public routes
│   ├── layout.tsx
│   ├── login/page.tsx
│   └── register/page.tsx
│
├── (main)/             # Route Group: Protected routes
│   ├── layout.tsx
│   ├── dashboard/
│   │   ├── page.tsx
│   │   ├── _components/    # Feature-specific components
│   │   ├── _hooks/         # Feature-specific hooks
│   │   └── _lib/           # Feature-specific loaders/actions
│   └── settings/
│       └── page.tsx
│
└── api/                # API routes (if needed)
```

**3. File Naming**
```
✅ page.tsx, layout.tsx, loading.tsx, error.tsx (lowercase)
✅ _components/, _hooks/, _lib/ (underscore prefix for private folders)
✅ (auth)/, (main)/ (parentheses for route groups)
✅ [productId]/ (square brackets for dynamic segments)
```

**4. Server vs Client Components**
```tsx
// ✅ Server Component (default) — for data fetching
// app/dashboard/page.tsx
import { getDashboardData } from "./_lib/dashboard.loader";

export default async function DashboardPage() {
  const data = await getDashboardData();
  return <DashboardDisplay data={data} />;
}

// ✅ Client Component — only when interactivity needed
// app/dashboard/_components/DashboardChart.tsx
"use client";

import { useState } from "react";

export function DashboardChart({ data }) {
  const [hoveredItem, setHoveredItem] = useState(null);
  // Interactive logic here...
}
```

**5. Server Actions Pattern**
```tsx
// ✅ Keep actions thin — business logic in services
// app/products/_lib/products.actions.ts
"use server";

import { productService } from "@/services/productService";

export async function createProduct(formData: FormData) {
  const data = validateProductInput(formData);
  return await productService.createProduct(data);
}

// ❌ DON'T put business logic in actions
```

**6. Data Loaders**
```tsx
// app/products/_lib/products.loader.ts
import { cache } from "react";
import { productService } from "@/services/productService";

export const loadProducts = cache(async (page: number) => {
  return await productService.getAll({ page });
});
```

**7. Common Patterns**
```tsx
// Loading skeleton
// app/dashboard/loading.tsx
export default function DashboardLoading() {
  return <DashboardSkeleton />;
}

// Error boundary (MUST be Client Component)
// app/dashboard/error.tsx
"use client";

export default function DashboardError({ error, reset }) {
  return (
    <div>
      <p>Error: {error.message}</p>
      <button onClick={reset}>Retry</button>
    </div>
  );
}

// Dynamic metadata
// app/products/[productId]/page.tsx
export async function generateMetadata({ params }) {
  const product = await loadProduct(params.productId);
  return {
    title: `${product.name} | MyShop`,
    description: product.description,
  };
}
```

---

#### `src/components/agents.md` — Shared Components

**1. Package Identity**
```
Global shared React components used across multiple features.
Atomic UI components, layout components, and cross-feature shared components.
```

**2. Folder Structure**
```
src/components/
├── ui/                 # Atomic/primitive components
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx
│   │   └── index.ts
│   ├── Input/
│   ├── Modal/
│   └── Card/
│
├── layout/             # Layout-level components
│   ├── Header.tsx
│   ├── Footer.tsx
│   ├── Sidebar.tsx
│   └── Navbar.tsx
│
└── shared/             # Cross-feature shared components
    ├── LoadingSpinner.tsx
    ├── ErrorBoundary.tsx
    └── PageWrapper.tsx
```

**3. Component Rules**
```tsx
// ✅ PascalCase for component files
// Button.tsx

/*
 * Component Name: Button
 * Description:    Primary button component with variants.
 * Props:
 *   - variant (string) — "primary" | "secondary" | "ghost"
 *   - size (string)    — "sm" | "md" | "lg"
 *   - isLoading (boolean) — Shows spinner when true
 */
export function Button({ variant = "primary", size = "md", isLoading, children }: ButtonProps) {
  return (
    <button className={cn(styles[variant], styles[size])}>
      {isLoading ? <Spinner /> : children}
    </button>
  );
}

// ✅ Always export from index.ts
// Button/index.ts
export { Button } from "./Button";
export type { ButtonProps } from "./Button";
```

**4. When to Create New Component**
```
Ask:
1. Is this used by 2+ features? → Put in src/components/
2. Is this used by 1 feature only? → Put in feature's _components/
3. Does a similar component exist? → EXTEND it, don't create new
```

---

#### `src/hooks/agents.md` — Custom Hooks

**1. Package Identity**
```
Global custom React hooks for shared stateful logic.
Used across multiple features.
```

**2. Hook Rules**
```tsx
// ✅ Hook naming: use[Feature/Capability]
// useDebounce.ts

/*
 * Hook Name:   useDebounce
 * Description: Debounces a value by the specified delay.
 * Parameters:  value (T), delay (number in ms)
 * Returns:     T — The debounced value
 */
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}
```

**3. Convex Integration**
```tsx
// ✅ Use Convex useQuery for reactive data
import { useQuery } from "convex/react";
import { api } from "@/convex/_generated/api";

export function useUser() {
  const user = useQuery(api.users.getCurrentUser);
  
  return {
    user,
    isLoading: user === undefined,
    isAuthenticated: user !== null,
  };
}

// ❌ Don't create separate hooks for the same data
// useUserName, useUserEmail, useUserRole → Just use useUser
```

---

### Phase 4: Pre-Implementation Checklist

**Before generating ANY agents.md file, verify:**

```markdown
## Code Pattern Checklist

### Convex
- [ ] All functions have args AND returns validators
- [ ] All queries use .withIndex()
- [ ] No floating promises (all awaited)
- [ ] Schema extensions use v.optional()
- [ ] Sensitive functions are internal

### Next.js
- [ ] Component files are PascalCase
- [ ] Non-component files are camelCase
- [ ] "use client" only where needed
- [ ] Private folders have underscore prefix
- [ ] Imports use @/ alias

### Optimization
- [ ] Checked for existing similar code
- [ ] Considered extending before creating
- [ ] Consolidated related queries
- [ ] Used computed properties where possible
```

---

## Output Format

Provide the files in this order:

1. **Analysis Summary** (from Phase 1)
2. **Root agents.md** (complete, ready to copy)
3. **Each Sub-Folder agents.md** (one at a time, with file path)

For each file, use this format:

```
---
File: `agents.md` (root)
---
[full content here]

---
File: `convex/agents.md`
---
[full content here]

---
File: `src/app/agents.md`
---
[full content here]
```

---

## Constraints & Quality Checks

Before generating, verify:

- [ ] Root agents.md is under 200 lines
- [ ] Root links to all sub-agents.md files
- [ ] Each sub-file has concrete examples (actual file paths from codebase)
- [ ] Commands are copy-paste ready
- [ ] No duplication between root and sub-files
- [ ] JIT hints use actual patterns from the codebase
- [ ] Every "✅ DO" has a real file example
- [ ] Every "❌ DON'T" references a real anti-pattern
- [ ] Pre-PR checks are single copy-paste commands
- [ ] Convex rules are prominently featured
- [ ] LEVER framework is mentioned for optimization guidance

---

## Reference Files

For deep dives, AI agents should reference:
- **Convex Rules:** `convex_rules.md` — Complete Convex best practices
- **Next.js Rules:** `nextjs_rules.md` — Complete Next.js conventions  
- **Optimization Rules:** `optimization_rules.md` — LEVER framework and decision trees
