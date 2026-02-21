# 🧠 Code Optimization Principles & Engineering Decision Framework
# Applies To: All code in this project (Next.js, Convex, shared libraries)
# Version: 1.0.0
# Last Updated: 2026-02-21

---

## 📑 Table of Contents

1.  [Core Philosophy](#1-core-philosophy)
2.  [The LEVER Framework](#2-the-lever-framework)
3.  [The Extended Thinking Process](#3-the-extended-thinking-process)
4.  [Pre-Implementation Checklist](#4-pre-implementation-checklist)
5.  [The Three-Pass Approach](#5-the-three-pass-approach)
6.  [Extend vs Create Decision Framework](#6-extend-vs-create-decision-framework)
7.  [Database Schema Optimization](#7-database-schema-optimization)
8.  [Query & API Optimization](#8-query--api-optimization)
9.  [Frontend State Optimization](#9-frontend-state-optimization)
10. [Convex-Specific Optimizations](#10-convex-specific-optimizations)
11. [Performance Optimization Rules](#11-performance-optimization-rules)
12. [Anti-Patterns to Avoid](#12-anti-patterns-to-avoid)
13. [Documentation Requirements for Extensions](#13-documentation-requirements-for-extensions)
14. [Success Metrics](#14-success-metrics)
15. [Review Checklist](#15-review-checklist)
16. [Real-World Case Study](#16-real-world-case-study)
17. [Cross-References](#17-cross-references)

---

## 1. Core Philosophy

> **"The best code is no code.**
> **The second best code is code that already exists and works."**

Every line of code you write is a **liability**, not an asset:

- It must be read and understood by every future developer.
- It must be tested, maintained, and updated when requirements change.
- It introduces new surface area for bugs.
- It adds to build times, bundle sizes, and cognitive load.

Before writing a single line, always ask:

```
1. Does this functionality already exist somewhere in this codebase?
2. Can I extend something that already exists?
3. Can I adapt an existing pattern instead of creating a new one?
4. Is the new code I am about to write reusable, or will it only be used once?
```

If you cannot answer all four questions with conviction, **stop and re-read
this document before proceeding.**

---

## 2. The LEVER Framework

Apply this mental framework to every feature request, bug fix, or refactor:

```
L — Leverage existing patterns first
E — Extend before creating anything new
V — Verify through reactivity and existing data flows
E — Eliminate every source of duplication
R — Reduce complexity at every decision point
```

### What Each Principle Means in Practice

| Principle | What to Ask | What to Do |
|-----------|-------------|------------|
| **Leverage** | What already exists that does something similar? | Map every existing file, function, and table that touches this domain |
| **Extend** | Can I add to what exists rather than create new? | Add fields, add return values, add computed properties |
| **Verify** | Does my change break existing consumers? | Check all callers; use `v.optional()` for additions |
| **Eliminate** | Am I duplicating logic that already exists? | Delete duplication ruthlessly; create a shared abstraction |
| **Reduce** | Is my solution simpler than the problem it solves? | If your solution is more complex than the problem, redesign |

---

## 3. The Extended Thinking Process

Before writing any code for a feature request, follow this decision tree. Do
not skip steps. Each step has a time budget — respect it.

```
┌─────────────────────────────────────────────────────────┐
│                  NEW FEATURE REQUEST                    │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
          ┌──────────────────────────┐
          │  Can existing code       │
          │  handle this as-is?      │
          └──────────────────────────┘
                 │            │
               YES             NO
                 │            │
                 ▼            ▼
        ┌──────────────┐  ┌──────────────────────────┐
        │ Use existing │  │ Can we modify/extend      │
        │ code. Done.  │  │ existing code to handle   │
        └──────────────┘  │ this?                     │
                          └──────────────────────────┘
                                 │            │
                               YES             NO
                                 │            │
                                 ▼            ▼
                        ┌──────────────┐  ┌──────────────────────┐
                        │ Extend the   │  │ Would the new code    │
                        │ existing     │  │ be reusable in 2 or   │
                        │ code.        │  │ more other places?    │
                        └──────────────┘  └──────────────────────┘
                                                 │            │
                                               YES             NO
                                                 │            │
                                                 ▼            ▼
                                        ┌──────────────┐  ┌──────────────┐
                                        │ Create a     │  │ STOP. Recon- │
                                        │ new reusable │  │ sider your   │
                                        │ abstraction. │  │ approach.    │
                                        └──────────────┘  └──────────────┘
```

---

## 4. Pre-Implementation Checklist

**Complete this before writing a single line of code.**
Time budget: 15–25 minutes. This investment prevents hours of wasted work.

### Step 1 — Pattern Recognition (10–15 minutes)

```markdown
## Existing Pattern Analysis

### Database
- [ ] What tables already exist that store related data?
- [ ] Can I add optional fields to an existing table instead of creating a new one?
- [ ] What indexes already exist that I could leverage?
- [ ] Are there existing foreign key relationships I can use?

### Backend Functions
- [ ] What queries already fetch related data?
- [ ] Can I extend an existing query's return value with new computed fields?
- [ ] What mutations already modify related documents?
- [ ] What internal functions already exist in this domain?

### Frontend
- [ ] What hooks already manage related state?
- [ ] What components already display similar information?
- [ ] What utility functions already process similar data?
- [ ] What context providers already cover this domain?

### State Management
- [ ] What store/context already manages related state?
- [ ] Can computed properties derived from existing data satisfy the requirement?
- [ ] Is there existing reactivity (Convex subscriptions) that automatically
      keeps this data up to date?
```

### Step 2 — Complexity Assessment (5–10 minutes)

Fill this out before writing code. Compare columns honestly.

```markdown
## Complexity Comparison

### Approach A: Create New (naive approach)
- Estimated lines of new code   : ___
- New files to create           : ___
- New database tables           : ___
- New API endpoints / functions : ___
- New state management logic    : ___
- Manual sync logic required    : ___

### Approach B: Extend Existing (optimized approach)
- Estimated lines of new code    : ___
- Files to modify                : ___
- Fields added to existing tables: ___
- Existing functions enhanced    : ___
- Convex reactivity handles sync : ___

## Decision Rule
If Approach B lines < 50% of Approach A lines → proceed with Approach B.
If Approach B lines > 75% of Approach A lines → reconsider; create new
may be appropriate.
```

---

## 5. The Three-Pass Approach

Never jump straight to implementation. Use three structured passes.

### Pass 1 — Discovery (No Code Written)

**Time budget: 10–15 minutes**

```markdown
Goal: Understand the full landscape of what already exists.

Actions:
1. Search the codebase for all files related to this domain.
2. Read every related query, mutation, hook, and component.
3. List all existing database tables and their fields that are relevant.
4. Write a short summary: "Here is what already exists that relates to X"
5. Identify the TOP 3 extension points (where you could add to existing code).

Output: A written list of existing code + 3 candidate extension points.
DO NOT write any implementation code yet.
```

### Pass 2 — Design (Minimal Code)

**Time budget: 10–15 minutes**

```markdown
Goal: Design the interface and data flow changes only.

Actions:
1. Update or sketch TypeScript type changes only (no implementation).
2. Design the new return shape of any extended query.
3. Plan which fields to add to which tables (all v.optional()).
4. Map out the data flow from database → query → hook → component.
5. Confirm no circular dependencies are introduced.

Output: Type definitions and a data flow diagram/notes.
DO NOT write handler logic yet.
```

### Pass 3 — Implementation (Optimized Code)

**Time budget: Remaining time**

```markdown
Goal: Implement with maximum reuse of existing code.

Actions:
1. Add fields to existing schemas (all optional).
2. Extend existing query handlers with new computed fields.
3. Extend existing hooks with new derived properties.
4. Extend existing components with conditional rendering.
5. Write ONLY the delta — not a full rewrite.
6. Document WHY each extension choice was made (see Section 13).

Output: The minimal change set that delivers the full feature.
```

---

## 6. Extend vs Create Decision Framework

When in doubt, use this scoring system. Add up points from each applicable row.

| Criteria | Extend Existing | Create New | Points |
|----------|-----------------|------------|--------|
| A similar data structure already exists | +3 | −3 | |
| Existing indexes cover the query pattern | +2 | −2 | |
| An existing query returns related data | +3 | −3 | |
| A UI component already shows similar info | +2 | −2 | |
| Extension requires fewer than 50 new lines | +3 | −3 | |
| Extension would cause circular dependencies | −5 | +5 | |
| Feature domain is significantly different | −3 | +3 | |
| More than 2 teams own the existing code | −2 | +2 | |
| Existing code is already complex/fragile | −2 | +2 | |
| New feature needs different caching strategy | −2 | +2 | |
| **TOTAL SCORE** | | | ___ |

**Score ≥ +5** → Extend existing code
**Score ≤ −5** → Create new implementation
**Score between −5 and +5** → Deeper analysis required; schedule a short
design review

---

## 7. Database Schema Optimization

### Rule 1: Extend Tables Before Creating New Ones

Every new table adds:
- Schema complexity and migration overhead
- Join requirements or denormalization choices
- Sync challenges when data must stay consistent across tables
- Additional read/write costs

```typescript
// ❌ Anti-Pattern: Create a separate tracking table
campaignTracking: defineTable({
  userId: v.id("users"),
  source: v.string(),
  medium: v.string(),
  campaign: v.string(),
  clickedAt: v.number(),
  convertedAt: v.optional(v.number()),
})
// Problems: requires joins, separate queries, sync logic,
// migration if users table changes

// ✅ Pattern: Add optional fields to the existing table
users: defineTable({
  // ... all existing fields unchanged ...

  // Campaign tracking — added as optional fields
  // Rationale: Data naturally belongs to the user. No joins needed.
  // Convex reactivity automatically keeps this consistent.
  campaignSource: v.optional(v.string()),
  campaignMedium: v.optional(v.string()),
  inviteCodeUsed: v.optional(v.string()),
  campaignClickedAt: v.optional(v.number()),
})
```

### Rule 2: All Schema Extensions Must Be Optional

```typescript
// ✅ Always add new fields as v.optional()
// This ensures:
// 1. Backward compatibility with existing documents
// 2. No migration required for existing data
// 3. Consumers can safely handle undefined values

users: defineTable({
  name: v.string(),                          // ← existing required field
  newFeatureField: v.optional(v.string()),   // ← new optional field
})
```

### Rule 3: Reuse Indexes Before Creating New Ones

```typescript
// ❌ Anti-Pattern: Create a new index for every query pattern
.index("by_campaign", ["campaignSource"])
.index("by_trial_status", ["subscriptionStatus"])
.index("by_campaign_and_status", ["campaignSource", "subscriptionStatus"])

// ✅ Pattern: Reuse existing indexes with filters
// Use the existing "by_subscription_status" index, then filter in memory
const trialUsers = await ctx.db
  .query("users")
  .withIndex("by_subscription_status", (q) =>
    q.eq("subscriptionStatus", "trialing")
  )
  .filter((q) => q.neq(q.field("campaignSource"), undefined))
  .collect();

// Note: .filter() after .withIndex() is acceptable — the index still
// does the heavy lifting. Only add a new index if the unindexed
// filter set is still > 1000 documents.
```

### Rule 4: Store Data in Its Most Logical Form

```typescript
// ❌ Anti-Pattern: UI-driven database design
// "We have a CampaignBadge component, so we need a campaigns table"

// ✅ Pattern: Store in the most natural location
// Store data where it semantically belongs.
// Use queries and computed properties to transform it for the UI.
// Let components derive their display values — do not pollute the DB
// schema with UI-specific shapes.
```

---

## 8. Query & API Optimization

### Rule 1: Extend Existing Queries Before Creating New Ones

```typescript
// ❌ Anti-Pattern: Multiple similar queries
export const getTrialUsers = query({ /* returns user + trial data */ });
export const getActiveTrials = query({ /* returns similar user + trial data */ });
export const getExpiringTrials = query({ /* also similar */ });
// Problems: 3x the maintenance, 3x the API surface, duplicate logic

// ✅ Pattern: One rich query with computed properties
export const getUserStatus = query({
  args: {},
  returns: v.union(
    v.object({
      // --- existing fields ---
      _id: v.id("users"),
      name: v.string(),
      email: v.string(),
      subscriptionStatus: v.string(),

      // --- extended fields (added without breaking existing consumers) ---
      isTrialing: v.boolean(),
      trialDaysRemaining: v.number(),
      isTrialFromCampaign: v.boolean(),
      campaignSource: v.optional(v.string()),
      shouldShowUpgradePrompt: v.boolean(),
    }),
    v.null()
  ),
  handler: async (ctx) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) return null;

    const user = await ctx.db
      .query("users")
      .withIndex("by_clerk_id", (q) => q.eq("clerkId", identity.subject))
      .unique();
    if (!user) return null;

    const trialDaysRemaining = calculateDaysRemaining(user.trialEndsAt);

    return {
      // Existing fields — unchanged
      _id: user._id,
      name: user.name,
      email: user.email,
      subscriptionStatus: user.subscriptionStatus,

      // Computed fields — no new table, no new query
      isTrialing: user.subscriptionStatus === "trialing",
      trialDaysRemaining,
      isTrialFromCampaign: Boolean(user.campaignSource),
      campaignSource: user.campaignSource,
      // Business logic computed from existing data
      shouldShowUpgradePrompt:
        user.subscriptionStatus === "trialing" && trialDaysRemaining <= 3,
    };
  },
});
```

### Rule 2: Consolidate Multiple Client Queries Into One

```typescript
// ❌ Anti-Pattern: Multiple useQuery calls for related data
const user = useQuery(api.users.getUser);
const subscription = useQuery(api.subscriptions.getSubscription);
const usage = useQuery(api.usage.getUsage);
const campaignData = useQuery(api.campaigns.getCampaignData);
// Problems: 4 round trips, 4 loading states, 4 error states,
// race conditions, stale data between them

// ✅ Pattern: One comprehensive query
const userStatus = useQuery(api.users.getUserStatus);
// Returns user + subscription + usage + campaign in one reactive call.
// One loading state. One error state. Always consistent.
```

### Rule 3: Computed Properties Over New Queries

```typescript
// ❌ Anti-Pattern: New query to derive something from existing data
export const getCampaignConversionValue = query({
  handler: async (ctx) => {
    const user = await getAuthenticatedUser(ctx);
    return user.subscriptionTier ? PRICES[user.subscriptionTier] * 12 : 0;
  },
});

// ✅ Pattern: Add computed property to the existing rich query
return {
  ...existingFields,
  // Derived entirely from data already being returned — no new query needed
  campaignConversionValue:
    user.subscriptionTier ? PRICES[user.subscriptionTier] * 12 : 0,
  isHighValueTrial:
    user.subscriptionTier === "creator" &&
    user.subscriptionStatus === "trialing",
};
```

---

## 9. Frontend State Optimization

### Rule 1: Extend Existing Hooks Before Creating New Ones

```typescript
// ❌ Anti-Pattern: Proliferating similar hooks
export function useTrialStatus() { /* ... */ }
export function useCampaignData() { /* ... */ }
export function useUserMetrics() { /* ... */ }
export function useSubscriptionStatus() { /* ... */ }
// Problems: 4x the imports, data duplicated in memory, potential
// inconsistencies between hook data

// ✅ Pattern: Extend the existing hook
export function useSubscription() {
  const userStatus = useQuery(api.users.getUserStatus);

  const enhancedData = useMemo(() => {
    if (!userStatus) return null;

    return {
      // --- existing properties ---
      ...userStatus,

      // --- new computed properties (added without breaking callers) ---
      // UI-level properties derived from existing data
      shouldShowTrialBanner:
        userStatus.isTrialing && userStatus.trialDaysRemaining <= 7,
      shouldShowUpgradeCta:
        userStatus.isTrialing && userStatus.trialDaysRemaining <= 3,
      trialUrgencyLevel:
        userStatus.trialDaysRemaining <= 1
          ? "critical"
          : userStatus.trialDaysRemaining <= 3
          ? "high"
          : "normal",
      campaignConversionLabel: userStatus.isTrialFromCampaign
        ? `Campaign: ${userStatus.campaignSource}`
        : null,
    };
  }, [userStatus]);

  return enhancedData;
}
```

### Rule 2: Extend Existing Components With Conditional Rendering

```typescript
// ❌ Anti-Pattern: Create a separate component for each feature variant
// CampaignSubscriptionStatus.tsx
// TrialSubscriptionStatus.tsx
// AdminSubscriptionStatus.tsx
// Each duplicates the core display logic

// ✅ Pattern: Add conditional rendering to the existing component
export function SubscriptionStatus() {
  const subscription = useSubscription();

  if (!subscription) return <SubscriptionSkeleton />;

  return (
    <div className="subscription-status">
      {/* ---- Existing UI — unchanged ---- */}
      <StatusBadge status={subscription.subscriptionStatus} />
      <PlanName tier={subscription.subscriptionTier} />

      {/* ---- New features — conditionally rendered ---- */}
      {subscription.isTrialing && (
        <TrialCountdown daysRemaining={subscription.trialDaysRemaining} />
      )}

      {subscription.isTrialFromCampaign && (
        <CampaignBadge source={subscription.campaignSource} />
      )}

      {subscription.shouldShowUpgradeCta && (
        <UpgradePrompt urgency={subscription.trialUrgencyLevel} />
      )}
    </div>
  );
}
```

### Rule 3: Leverage Reactivity — Eliminate Manual Sync

```typescript
// ❌ Anti-Pattern: Manual state management when Convex reactivity exists
const [userData, setUserData] = useState(null);
const [trialData, setTrialData] = useState(null);

useEffect(() => {
  // Polling, manual refetching, or subscription management
  const interval = setInterval(async () => {
    const data = await fetchUserData();
    setUserData(data);
  }, 5000);
  return () => clearInterval(interval);
}, []);
// Problems: stale data, race conditions, complex cleanup, manual sync

// ✅ Pattern: Let Convex reactivity handle everything
const userStatus = useQuery(api.users.getUserStatus);
// This is a live subscription. When data changes in the database,
// every component using this hook automatically re-renders.
// No polling. No manual sync. No stale data. No cleanup needed.
```

---

## 10. Convex-Specific Optimizations

### Optimize the Public/Internal Split

```typescript
// ✅ Keep the public surface lean — internal functions do the heavy work

// Public: returns only what the client needs
export const getUserStatus = query({
  args: {},
  returns: v.union(v.object({ /* client-safe fields only */ }), v.null()),
  handler: async (ctx) => {
    const rawUser = await ctx.runQuery(internal.users.getRawUser, {});
    return transformForClient(rawUser); // strip sensitive fields
  },
});

// Internal: has access to all fields including sensitive ones
export const getRawUser = internalQuery({
  args: {},
  returns: v.union(v.object({ /* all fields including sensitive */ }), v.null()),
  handler: async (ctx) => {
    // full document access
  },
});
```

### Optimize Mutation Atomicity

```typescript
// ✅ Keep related writes in one mutation — avoid split mutations
// that create race conditions

// ❌ Anti-Pattern: Two separate mutations for one logical operation
// mutation 1: createPost
// mutation 2: createNotification
// Problem: if mutation 2 fails, post exists but no notification is created

// ✅ Pattern: One atomic mutation for the whole operation
export const createPostWithNotification = mutation({
  args: { content: v.string() },
  returns: v.id("posts"),
  handler: async (ctx, args) => {
    const postId = await ctx.db.insert("posts", { content: args.content });
    await ctx.db.insert("notifications", { postId, type: "new_post" });
    // If either write fails, BOTH are rolled back automatically
    return postId;
  },
});
```

### Optimize Scheduled Actions

```typescript
// ✅ Schedule slow work — do not block the mutation waiting for it
export const sendMessage = mutation({
  args: { content: v.string(), channelId: v.id("channels") },
  returns: v.null(),
  handler: async (ctx, args) => {
    // Fast: write to DB immediately — UI updates right away
    await ctx.db.insert("messages", {
      content: args.content,
      channelId: args.channelId,
    });
    // Fast: schedule the slow work (AI, email, etc.)
    // Do NOT inline the action — schedule it instead
    await ctx.scheduler.runAfter(
      0,
      internal.messages.processWithAi,
      { channelId: args.channelId }
    );
    return null;
  },
});
```

---

## 11. Performance Optimization Rules

### Rule 1: Indexes First, Filters Second

```typescript
// Performance hierarchy (fastest to slowest):
// 1. .withIndex() — all fields matched exactly         → O(log n)
// 2. .withIndex() — partial match + .filter()          → O(log n + k)
// 3. Full table scan + .filter()                       → O(n) NEVER in production

// ✅ Always lead with the most selective index
const result = await ctx.db
  .query("messages")
  .withIndex("by_channel_and_author", (q) =>
    q.eq("channelId", args.channelId).eq("authorId", args.authorId)
  ) // ← index narrows to a small set first
  .filter((q) => q.neq(q.field("isDeleted"), true)) // ← small in-memory filter
  .order("desc")
  .take(20);
```

### Rule 2: Batch Writes — Never Loop Awaits

```typescript
// ❌ Anti-Pattern: Sequential awaits in a loop (O(n) round trips)
for (const item of items) {
  await ctx.db.patch(item._id, { processed: true });
}
// 1000 items = 1000 sequential round trips to the database

// ✅ Pattern: Parallel writes with Promise.all
await Promise.all(
  items.map((item) => ctx.db.patch(item._id, { processed: true }))
);
// 1000 items = 1 batch of parallel writes

// ✅ For very large sets, chunk the batches to avoid memory pressure
const BATCH_SIZE = 100;
for (let i = 0; i < items.length; i += BATCH_SIZE) {
  const batch = items.slice(i, i + BATCH_SIZE);
  await Promise.all(batch.map((item) => ctx.db.patch(item._id, updates)));
}
```

### Rule 3: Use .take(n) Over .collect() for Bounded Lists

```typescript
// ❌ .collect() fetches ALL matching documents into memory
const allMessages = await ctx.db
  .query("messages")
  .withIndex("by_channel", (q) => q.eq("channelId", args.channelId))
  .collect(); // could fetch 10,000+ documents

// ✅ .take(n) fetches only what you need
const recentMessages = await ctx.db
  .query("messages")
  .withIndex("by_channel", (q) => q.eq("channelId", args.channelId))
  .order("desc")
  .take(20); // always fetches exactly 20 documents

// ✅ For larger lists shown in the UI, always paginate
.paginate(args.paginationOpts) // fetches one page at a time
```

### Rule 4: Eliminate Waterfall Queries

```typescript
// ❌ Anti-Pattern: Waterfall queries (each awaits the previous)
const user = await ctx.db.get(args.userId);
const subscription = await ctx.db.get(user.subscriptionId);  // waits for user
const plan = await ctx.db.get(subscription.planId);           // waits for subscription
// Total time = time(user) + time(subscription) + time(plan)

// ✅ Pattern: Parallel queries for independent data
const [user, settings, notifications] = await Promise.all([
  ctx.db.get(args.userId),
  ctx.db
    .query("settings")
    .withIndex("by_user", (q) => q.eq("userId", args.userId))
    .unique(),
  ctx.db
    .query("notifications")
    .withIndex("by_user", (q) => q.eq("userId", args.userId))
    .take(5),
]);
// Total time = max(time(user), time(settings), time(notifications))
```

### Rule 5: Performance Targets

| Operation | Green Target | Yellow | Red — Investigate |
|-----------|-------------|--------|-------------------|
| Query completion time | < 100ms | 100–300ms | > 300ms |
| Mutation completion time | < 50ms | 50–150ms | > 150ms |
| Documents returned per query | < 100 | 100–1000 | > 1000 |
| Action completion time | < 5s | 5–10s | > 10s |
| Hook re-renders per interaction | ≤ 2 | 3–4 | > 4 |
| New files per feature | 0–2 | 3–4 | > 4 |

---

## 12. Anti-Patterns to Avoid

### Anti-Pattern 1: The "Just One More Table" Trap

**Symptom:** Every new feature gets its own table.

**Problem:** Each new table means:
- Joins or denormalization decisions
- Sync logic to keep tables consistent
- More indexes to maintain and pay for
- Migration overhead on every schema change
- A data model that becomes harder to understand over time

**Fix:** Ask: "Can this data live as optional fields in an existing table?"

---

### Anti-Pattern 2: The "Similar But Different" Excuse

**Symptom:** "I know `getUserStatus` exists, but my case is slightly different
so I will create `getUserTrialStatus`."

**Problem:** Two functions doing 80% the same work:
- Double the maintenance burden
- Double the surface area for bugs
- Future developers unsure which to use
- The two implementations drift apart over time

**Fix:** Extend `getUserStatus` to return the additional data.
Use `v.optional()` on new return fields so existing consumers are unaffected.

---

### Anti-Pattern 3: The "UI Drives Database" Mistake

**Symptom:** "We have a `CampaignBadge` component, so we need a `campaigns` table."

**Problem:** UI requirements change constantly. If your database schema mirrors
your UI structure, every redesign requires a data migration.

**Fix:** Store data in its most semantically correct location.
Use queries and computed properties to transform it for the UI.
Components should derive their display values from data — not demand that
the database matches their shape.

---

### Anti-Pattern 4: The "Polling Instead of Reactivity" Mistake

**Symptom:**
```typescript
setInterval(async () => {
  const data = await fetchFromConvex();
  setData(data);
}, 3000);
```

**Problem:** Stale data between polls, race conditions, battery drain on
mobile devices, unnecessary server load, complex cleanup code.

**Fix:** Use `useQuery(api.yourFunction)`. It is a live reactive subscription.
Data updates automatically the moment it changes in the database.
No interval. No cleanup. No stale data.

---

### Anti-Pattern 5: The "Hook for Every Concern" Explosion

**Symptom:** `useTrialStatus`, `useCampaignData`, `useUserMetrics`,
`useSubscriptionFeatures`, `useUsageLimits` — all consuming the same
underlying data query.

**Problem:** The same data fetched multiple times per render, inconsistency
between hooks during loading states, harder to trace the data flow, more
files to maintain.

**Fix:** One hook per domain. Add computed properties to it as the domain
grows. `useSubscription()` should return everything subscription-related.
When it gets too large, split by feature — not by data field.

---

### Anti-Pattern 6: The "Floating Promise" Bug

**Symptom:**
```typescript
// No await — these may silently fail or execute out of order
ctx.db.patch(userId, { lastSeen: Date.now() });
ctx.scheduler.runAfter(0, internal.emails.send, {});
```

**Problem:** Convex functions that are not awaited can fail silently, execute
out of order, or not execute at all. ESLint will not always catch this.

**Fix:** Always `await` every `ctx.db` call, every `ctx.scheduler` call,
and every `ctx.runQuery` / `ctx.runMutation` / `ctx.runAction` call.

```typescript
// ✅ Correct
await ctx.db.patch(userId, { lastSeen: Date.now() });
await ctx.scheduler.runAfter(0, internal.emails.send, {});
```

---

### Anti-Pattern 7: The "Action as a Transaction" Mistake

**Symptom:**
```typescript
export const processOrder = action({
  handler: async (ctx, args) => {
    await ctx.runMutation(internal.orders.chargeCard, args);
    await ctx.runMutation(internal.orders.updateInventory, args);
    await ctx.runMutation(internal.orders.sendConfirmation, args);
    // If step 3 fails, steps 1 and 2 already committed — no rollback
  },
});
```

**Problem:** Actions are NOT transactions. Each `runMutation` call is its
own separate commit. If step 3 fails, steps 1 and 2 have already been
permanently written.

**Fix:** Put all related writes into a single mutation so they are atomic.
Use actions only for side effects that cannot be transactional by nature
(external API calls, email sending, etc.).

---

## 13. Documentation Requirements for Extensions

Whenever you extend existing code, document the **why**, not the **what**.
The code already shows what it does — comments should explain the reasoning.

### Inline Documentation Format for Extensions

```typescript
export const getUserStatus = query({
  handler: async (ctx) => {
    // ... existing implementation unchanged above this line ...

    return {
      // ---- EXISTING FIELDS — do not remove or rename ----
      _id: user._id,
      name: user.name,
      subscriptionStatus: user.subscriptionStatus,

      // ---- EXTENSION: Campaign Tracking ----
      // Author:  Yasir Khan
      // Date:    2026-02-21
      // Ticket:  PROJ-1234
      // WHY:     Added campaign fields here instead of creating a separate
      //          campaignTracking table. Campaign data is 1:1 with the user
      //          so a separate table would only add join complexity.
      //          Fields are v.optional() so all existing callers are
      //          unaffected. Convex reactivity handles propagation.
      campaignSource: user.campaignSource,
      isTrialFromCampaign: Boolean(user.campaignSource),
    };
  },
});
```

### Extension Decision Log

For significant architecture decisions, add a block comment at the top of
the file or function where the decision was made:

```typescript
/*
 * Extension Decision Log
 * ──────────────────────────────────────────────────────────────────────
 * Date:        2026-02-21
 * Author:      Yasir Khan
 * Feature:     Campaign trial tracking
 * Ticket:      PROJ-1234
 *
 * Decision:    Extend users table + getUserStatus query
 *
 * Considered:  Creating a separate campaignTracking table with its own
 *              queries, mutations, and hooks.
 *
 * Rationale:   Campaign data is a 1:1 relationship with users. A separate
 *              table would require joins, sync mutations, and 3+ new files.
 *              Extending the existing table saved approximately 900 lines
 *              of code while maintaining the same functionality.
 *              Convex reactivity handles all update propagation automatically
 *              with no additional sync logic.
 *
 * Result:      87% reduction in code vs the naive approach.
 * ──────────────────────────────────────────────────────────────────────
 */
```

### TODO and FIXME Format

```typescript
// TODO  [Yasir Khan | 2026-02-21 | PROJ-1234]: Replace with RAG pipeline.
// FIXME [Yasir Khan | 2026-02-21 | PROJ-1235]: Rate limiting not enforced.
```

---

## 14. Success Metrics

Use these metrics to evaluate whether your optimization was successful.
Fill this table out in your PR description for every new feature.

| Metric | Green ✅ | Yellow ⚠️ | Red ❌ |
|--------|---------|----------|-------|
| Code reduction vs naive approach | > 50% | 25–50% | < 25% |
| Existing patterns reused | > 70% | 50–70% | < 50% |
| New files created per feature | 0–2 | 3–4 | > 4 |
| New database tables created | 0 | 1 | > 1 |
| New indexes required | 0 | 1 | > 1 |
| Estimated vs actual implementation time | < 50% of estimate | 50–75% | > 75% |
| Manual sync logic required | None | Minimal | Yes |
| Consumer breakage from changes | Zero | Compile-time only | Runtime errors |

---

## 15. Review Checklist

Run this checklist on every PR that introduces new data or feature logic.
All items must be checked before requesting a review.

### Schema
- [ ] Extended existing tables rather than creating new ones
- [ ] All new fields added as `v.optional()` for backward compatibility
- [ ] No redundant indexes created (reused existing ones with filters)
- [ ] Documents are flat — no deeply nested arrays of objects in a document

### Backend Functions (Convex)
- [ ] Extended existing queries with new computed fields where possible
- [ ] No duplicate query logic created across similar functions
- [ ] Sensitive operations use `internalMutation` / `internalAction`
- [ ] Slow work scheduled via `ctx.scheduler` — not blocking the mutation
- [ ] All promises are awaited — no floating promises anywhere
- [ ] Related writes are in one atomic mutation — not split across multiple

### Frontend (Next.js)
- [ ] Extended existing hooks with new computed properties
- [ ] Extended existing components with conditional rendering
- [ ] Convex reactivity used — no manual polling or sync logic
- [ ] Multiple related `useQuery` calls consolidated where appropriate
- [ ] No duplicate state management logic for the same domain

### General
- [ ] Extension decision documented inline (why, not what)
- [ ] No circular dependencies introduced
- [ ] Backward compatibility maintained — existing consumers unaffected
- [ ] Code reduction ≥ 50% vs naive implementation
- [ ] Success metrics table filled out in PR description
- [ ] TODO/FIXME comments include author, date, and ticket number

---

## 16. Real-World Case Study

### Trial User Upgrade Flow Optimization

This case study is the origin of many principles in this document.
Reference it when teammates question why we extend instead of create.

#### The Naive Approach — 1,050 Lines

```
New Database Tables (4):
├── campaignTracking   (userId, source, medium, clickedAt, convertedAt, ...)
├── trialExtensions    (userId, reason, grantedAt, expiresAt, ...)
├── conversionEvents   (userId, fromTier, toTier, revenue, ...)
└── upgradeAttempts    (userId, attemptedAt, failureReason, ...)

New Queries (10+):
├── getTrialStatus
├── getCampaignData
├── getConversionHistory
├── getUpgradeAttempts
├── getTrialExtensions
└── ... 5+ more similar queries

New Hooks (5):
├── useTrialStatus
├── useCampaignData
├── useConversionTracking
├── useUpgradeFlow
└── useTrialExtension

New Components (5+):
└── A separate component tree for every state variant

Manual Sync Logic:
└── 8 mutations to keep 4 tables consistent with each other
```

#### The Optimized Approach — 140 Lines

```
Schema Changes:
└── users table — 11 optional fields added
    (campaignSource, campaignMedium, inviteCodeUsed, trialEndsAt, ...)

Query Changes:
└── getUserStatus — extended with 8 new computed properties
    (isTrialing, trialDaysRemaining, isTrialFromCampaign, ...)

Hook Changes:
└── useSubscription — extended with 4 UI-level computed properties
    (shouldShowTrialBanner, trialUrgencyLevel, ...)

Component Changes:
└── SubscriptionStatus — 3 conditional render blocks added
    (TrialCountdown, CampaignBadge, UpgradePrompt)

Sync Logic:
└── None — Convex reactivity handles all propagation automatically
```

#### Results

| Measure | Naive | Optimized | Reduction |
|---------|-------|-----------|-----------|
| Total lines of code | 1,050 | 140 | **87%** |
| New database tables | 4 | 0 | **100%** |
| New queries | 10+ | 0 | **100%** |
| New hooks | 5 | 0 | **100%** |
| Manual sync mutations | 8 | 0 | **100%** |
| Estimated implementation time | ~3 days | ~4 hours | **~80%** |
| Bug surface area | High | Minimal | — |
| Maintenance burden | High | Minimal | — |

---

## 17. Cross-References

This file governs the **thinking and decision-making process** before
implementation. For implementation-specific coding standards, refer to:

| File | When to Open |
|------|-------------|
| `optimization_rules.mdc` (this file) | Before starting ANY new feature or significant change |
| `convex_rules.mdc` | While writing or reviewing Convex backend code |
| `nextjs_rules.mdc` | While writing or reviewing Next.js frontend code |

### Recommended Workflow

```
1. New ticket arrives
         │
         ▼
2. Open optimization_rules.mdc
   → Complete Pre-Implementation Checklist (Section 4)
   → Run the Extended Thinking Process (Section 3)
   → Score Extend vs Create (Section 6)
         │
         ▼
3. Write backend code
   → Follow convex_rules.mdc
         │
         ▼
4. Write frontend code
   → Follow nextjs_rules.mdc
         │
         ▼
5. Before opening PR
   → Run Review Checklist (Section 15)
   → Fill out Success Metrics (Section 14)
```

---

*"Every line of code is a liability. The best feature is one that requires
no new code — just better use of what already exists."*
```