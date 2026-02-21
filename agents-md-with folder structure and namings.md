# Analyze This Codebase and Generate a Hierarchical agents.md Structure

## Context & Principles

You are going to help me create a **hierarchical agents.md system** for this
codebase. This is critical for AI coding agents to work efficiently with
minimal token usage.

### Core Principles

1. **Root agents.md is LIGHTWEIGHT** — Only universal guidance, links to sub-files
2. **Nearest-wins hierarchy** — Agents read the closest agents.md to the file
   being edited
3. **JIT (Just-In-Time) indexing** — Provide paths/globs/commands, NOT full content
4. **Token efficiency** — Small, actionable guidance over encyclopedic documentation
5. **Sub-folder agents.md files have MORE detail** — Specific patterns, examples,
   commands
6. **Structure before code** — Always establish the correct folder structure and
   apply all naming conventions BEFORE creating any files or modifying any code
7. **Conventions are non-negotiable** — Every file name, folder name, function
   name, variable name, component name, and comment block must follow the
   conventions defined in Phase 0 before anything is written

---

## Phase 0: Structure & Conventions First (ALWAYS RUN BEFORE ANYTHING ELSE)

> ⚠️ **This phase is mandatory.** No files may be created, no code may be
> written, and no existing files may be modified until Phase 0 is complete
> and the structure + conventions have been confirmed.

### Step 0.1 — Scaffold the Hybrid Folder Structure

Before touching any logic, scaffold the complete **Next.js 16 App Router
hybrid folder structure** (layer-based + feature-based). Create ALL critical
folders even if the business logic for them does not exist yet. Empty folders
must include a `.gitkeep` file so they are tracked by Git.

The required structure to scaffold is:

```
src/
├── app/
│   ├── (auth)/
│   │   ├── login/
│   │   └── register/
│   ├── (main)/
│   │   ├── dashboard/
│   │   │   ├── _components/
│   │   │   ├── _hooks/
│   │   │   └── _lib/
│   │   ├── settings/
│   │   │   ├── _components/
│   │   │   └── _lib/
│   │   └── [feature]/        ← repeat this pattern for each feature
│   │       ├── _components/
│   │       ├── _hooks/
│   │       └── _lib/
│   └── api/
│       └── [domain]/
├── components/
│   ├── ui/
│   ├── layout/
│   └── shared/
├── hooks/
├── services/
├── lib/
├── utils/
├── types/
├── stores/
├── styles/
└── config/
```

Rules for scaffolding:
- NEVER create a folder without first checking whether it belongs to the
  global layer (`components/`, `hooks/`, `services/`, etc.) or to a
  feature (`app/(main)/[feature]/_components/`, etc.)
- NEVER create a folder inside `app/` without an underscore prefix
  (`_components/`, `_hooks/`, `_lib/`) UNLESS it is a Next.js route segment
- Use route groups `(auth)/` and `(main)/` from the start — never add them
  retroactively
- ALWAYS create `convex/` at the project root if Convex is part of the stack,
  alongside `src/`

---

### Step 0.2 — Naming Conventions (Apply Before Every File or Code Change)

> Every agent, at every step, must verify naming conventions BEFORE creating
> or modifying any file. This check is not optional.

#### 📁 Folder Naming Rules

| Rule | ✅ Good | ❌ Bad |
|------|---------|--------|
| Always lowercase | `components/` | `Components/` |
| Use plural for collections | `hooks/`, `services/` | `hook/`, `service/` |
| Use meaningful names | `auth/`, `dashboard/` | `folder1/`, `stuff/` |
| Underscore prefix for private app folders | `_components/` | `components/` inside `app/` |
| Parentheses for route groups | `(auth)/` | `auth-group/` |

#### 📄 File Naming Rules

| File Type | Convention | ✅ Good | ❌ Bad |
|-----------|------------|---------|--------|
| React components | PascalCase `.tsx` | `UserCard.tsx` | `user-card.tsx` |
| Page files | PascalCase `.tsx` | `LoginPage.tsx` | `login_page.tsx` |
| Non-component files | camelCase `.ts` | `authService.ts` | `AuthService.ts` |
| Hooks | camelCase `.ts` | `useDebounce.ts` | `UseDebounce.ts` |
| Utility/helper files | camelCase `.ts` | `formatDate.ts` | `FormatDate.ts` |
| Next.js reserved files | lowercase | `page.tsx`, `layout.tsx` | `Page.tsx` |
| Test files | same name + `.test` | `Button.test.tsx` | `button-test.tsx` |
| Barrel files | lowercase | `index.ts` | `Index.ts` |

#### 🧩 Component Naming Rules

```typescript
// ✅ Good — PascalCase, describes what it renders
function LoginForm() {}
function UserCard() {}
function DashboardChart() {}
const ProductList = () => {};

// ❌ Bad
function user_card() {}
function loginform() {}
const productlist = () => {};
```

#### 📝 Variable Naming Rules

```typescript
// ✅ Good — camelCase
let userName: string;
let totalPrice: number;
let currentUserId: string;

// ✅ Boolean — must start with is, has, can, or should
let isLoading: boolean;
let hasError: boolean;
let canEdit: boolean;
let shouldRedirect: boolean;

// ✅ Constants — SCREAMING_SNAKE_CASE
const MAX_RETRY_ATTEMPTS = 3;
const API_BASE_URL = "https://api.example.com";

// ❌ Bad
let Total_Price: number;
let loading: boolean;
const maxRetries = 3;
```

#### ⚙️ Function Naming Rules

```typescript
// ✅ Good — camelCase, verb-noun pattern
function getUserData() {}
function fetchProducts() {}
function calculateTotalPrice() {}
function handleFormSubmit() {}    // event handlers → handle prefix
function formatDateToISO() {}     // transformers → descriptive verb
async function fetchUserById() {} // async fetchers → fetch or get prefix

// ❌ Bad — too vague, wrong case, or undescriptive
function GetUserData() {}  // PascalCase is for components only
function doTask() {}
function process() {}
function calc() {}
```

#### 🗂️ Next.js Page Function Naming Rules

```typescript
// ✅ App Router page — default export, PascalCase, descriptive name
export default function DashboardPage() {}
export default function LoginPage() {}
export default function ProductDetailPage() {}

// ✅ Layout — always named Layout with the segment prefix
export default function DashboardLayout({ children }: { children: React.ReactNode }) {}

// ✅ Loading and Error — descriptive
export default function DashboardLoading() {}
export default function DashboardError() {}

// ❌ Bad
export default function Page() {}    // too generic
export default function page() {}    // lowercase
export default function Index() {}   // legacy pattern
```

---

### Step 0.3 — Code Documentation Rules (Apply to Every File Created or Modified)

> Every file and every exported function must include the documentation
> blocks below. This is enforced before any PR is considered ready.

#### 📋 File Header Block

Every file must begin with this block as the very first content:

```typescript
/*
 * File Name:     authService.ts
 * Description:   Handles all authentication-related API calls including
 *                login, logout, token refresh, and session validation.
 * Author:        [Author Name]
 * Created Date:  YYYY-MM-DD
 * Last Modified: YYYY-MM-DD
 */
```

#### 🔧 Function / Hook Documentation Block

Every exported function, hook, or server action must have this block
immediately above its definition:

```typescript
/*
 * Function Name: getUserData
 * Description:   Fetches user data from the API by user ID.
 *                Returns null if the user does not exist.
 * Parameters:
 *   - userId (string) — The unique identifier of the user
 * Returns:       Promise<User | null>
 */
export async function getUserData(userId: string): Promise<User | null> {
  // implementation
}
```

#### 🧩 React Component Documentation Block

For React components, the block must describe the component's purpose
and every prop it accepts:

```typescript
/*
 * Component Name: UserCard
 * Description:    Displays a user's profile card with their avatar,
 *                 display name, and role badge.
 * Props:
 *   - user       (User)              — The user object to display
 *   - isLoading  (boolean)           — Shows a skeleton loader if true
 *   - onEdit     (() => void)        — Callback when the Edit button is clicked
 *   - className  (string, optional)  — Additional CSS classes
 */
export function UserCard({ user, isLoading, onEdit, className }: UserCardProps) {
  // implementation
}
```

#### 🪝 Custom Hook Documentation Block

```typescript
/*
 * Hook Name:   useDebounce
 * Description: Debounces a changing value by the specified delay in
 *              milliseconds. Prevents excessive API calls on rapid input.
 * Parameters:
 *   - value (T)        — The value to debounce
 *   - delay (number)   — Debounce delay in milliseconds
 * Returns:     T — The debounced value, updated after the delay elapses
 */
export function useDebounce<T>(value: T, delay: number): T {
  // implementation
}
```

#### 💬 Inline Comment Rules

```typescript
// ✅ Good — explains WHY, not what
// Delay by 300ms to avoid hitting the search API rate limit
const debouncedSearch = useDebounce(searchQuery, 300);

// ✅ Good — explains a non-obvious decision
// We use replace() instead of patch() here because the entire subscription
// object must be replaced atomically to avoid partial state
await ctx.db.replace(subscriptionId, newSubscription);

// ❌ Bad — restates what the code already says
// Set the delay to 300
const delay = 300;

// ❌ Bad — no context
// Handle error
catch (error) { throw error; }
```

#### 🔖 TODO and FIXME Comment Format

```typescript
// TODO  [Author Name | YYYY-MM-DD | TICKET-123]: Migrate to React Query v5.
// FIXME [Author Name | YYYY-MM-DD | TICKET-456]: Breaks on Safari 16 — investigate.
```

---

## Phase 1: Repository Analysis

After completing Phase 0, analyze the codebase and provide:

1. **Repository type**: Monorepo, multi-package, or single project?
2. **Primary technology stack**: Languages, frameworks, key tools
3. **Major directories** that should each have their own agents.md:
   - Apps (e.g., `apps/web`, `apps/api`)
   - Services (e.g., `services/auth`, `services/transcribe`)
   - Packages/libs (e.g., `packages/ui`, `packages/shared`)
   - Workers/jobs (e.g., `workers/queue`, `workers/cron`)
4. **Build system**: pnpm/npm/yarn workspaces? Turborepo? Or simple?
5. **Testing setup**: Jest, Vitest, Playwright? Where are tests located?
6. **Key patterns to document**:
   - Code organization patterns
   - Critical files that serve as examples
   - Anti-patterns already present in the codebase

Present this as a **structured map** before generating any agents.md files.

---

## Phase 2: Generate Root agents.md

Create a **lightweight root agents.md** (~100–200 lines max) that includes:

### Required Sections

**1. Project Snapshot** (3–5 lines)
- Repo type (monorepo/simple)
- Primary tech stack
- Note that sub-packages have their own agents.md files

**2. Root Setup Commands** (5–10 lines)
- Install dependencies
- Build all
- Typecheck all
- Test all

**3. Universal Conventions** (5–10 lines)
- TypeScript strict mode
- Prettier + ESLint enforced
- Commit format (Conventional Commits)
- Branch strategy
- PR requirements

**4. Naming Convention Reference** (3–5 lines)
- Reference to Phase 0 conventions above
- Note that ALL conventions in Phase 0 are enforced on every file

**5. Security & Secrets** (3–5 lines)
- Never commit tokens or `.env.local`
- Where secrets go
- PII handling if applicable

**6. JIT Index — Directory Map** (10–20 lines)
```markdown
## JIT Index (what to open, not what to paste)

### Package Structure
- Web UI: `apps/web/` → [see apps/web/agents.md](apps/web/agents.md)
- API: `apps/api/` → [see apps/api/agents.md](apps/api/agents.md)
- Convex backend: `convex/` → [see convex/agents.md](convex/agents.md)
- Shared packages: `packages/**/` → [see packages/README.md]

### Quick Find Commands
- Find a component:     `rg -n "export function .*" src/components`
- Find a hook:          `rg -n "export.*use[A-Z]" src/hooks`
- Find a service:       `rg -n "export" src/services`
- Find a page:          `rg -n "export default function.*Page" src/app`
- Find a Convex query:  `rg -n "export const.*= query" convex/`
- Find a type:          `rg -n "export type|export interface" src/types`
```

**7. Definition of Done** (3–5 lines)
- All naming conventions from Phase 0 must be applied
- All file header blocks must be present
- All exported functions/components must have documentation blocks
- All checks must pass before a PR is opened

---

## Phase 3: Generate Sub-Folder agents.md Files

For EACH major package/directory identified in Phase 1, create a detailed
agents.md that includes:

### Required Sections

**1. Package Identity** (2–3 lines)
- What this package/app/service does
- Primary tech/framework for this package

**2. Setup & Run** (5–10 lines)
- Install command (if different from root)
- Dev server command
- Build command
- Test command
- Lint/typecheck commands

**3. Folder Structure for This Package** (10–15 lines)

Document the exact folder structure for this sub-package, following the
hybrid approach. Example for `apps/web`:

```
src/
├── app/
│   ├── (auth)/login/         ← Public login route
│   ├── (main)/dashboard/     ← Protected dashboard route
│   │   ├── _components/      ← Dashboard-specific components (private)
│   │   ├── _hooks/           ← Dashboard-specific hooks (private)
│   │   └── _lib/             ← Dashboard loaders and actions (private)
│   └── api/                  ← API route handlers
├── components/ui/             ← Global atomic components
├── components/layout/         ← Global layout components
├── hooks/                     ← Global custom hooks
├── services/                  ← Global API service layer
├── lib/                       ← Third-party wrappers
├── utils/                     ← Pure utility functions
└── types/                     ← Global TypeScript types
```

**4. Patterns & Conventions for This Package** (10–20 lines)

```markdown
## Naming Conventions (enforced — see Phase 0 for full rules)

### Folders
- ✅ DO:  `src/components/ui/` — lowercase, plural
- ❌ DON'T: `src/Components/UI/` — never PascalCase folders

### Files
- ✅ DO:  `UserCard.tsx` for components
- ✅ DO:  `authService.ts` for services
- ❌ DON'T: `usercard.tsx`, `AuthService.ts`

### Components
- ✅ DO:  `export function UserCard() {}` — See `src/components/ui/Button/Button.tsx`
- ❌ DON'T: `export function user_card() {}`

### Pages
- ✅ DO:  `export default function DashboardPage() {}`
- ❌ DON'T: `export default function Page() {}`

### Functions
- ✅ DO:  `getUserData()`, `handleFormSubmit()`, `fetchProducts()`
- ❌ DON'T: `GetUser()`, `doTask()`, `process()`

### Variables
- ✅ DO:  `let userName`, `let isLoading`, `const MAX_ITEMS = 10`
- ❌ DON'T: `let UserName`, `let loading`, `const maxItems = 10`

## File Documentation (enforced on every file)
- Every new file must begin with the file header block (see Phase 0.3)
- Every exported function must have the function documentation block
- Every React component must have the component documentation block
- Inline comments explain WHY, not what
```

**5. Touch Points / Key Files** (5–10 lines)

Point to the most important files to understand this package:
```
- Auth logic:      `src/auth/provider.tsx`
- API client:      `src/lib/apiClient.ts`
- Types:           `src/types/index.ts`
- Global styles:   `src/styles/variables.css`
- Config:          `src/config/constants.ts`
- Route map:       `src/config/routes.ts`
```

**6. JIT Index Hints** (5–10 lines)

```markdown
- Find a component:        `rg -n "export function" src/components`
- Find a hook:             `rg -n "export.*use[A-Z]" src/hooks`
- Find a page function:    `rg -n "export default function.*Page" src/app`
- Find a service:          `rg -n "export" src/services`
- Find a server action:    `rg -n "\"use server\"" src/app`
- Find a type/interface:   `rg -n "export type|export interface" src/types`
- Find all file headers:   `rg -n "File Name:" src/`
```

**7. Common Gotchas** (3–5 lines)
- "All new `app/` folders need an underscore prefix to stay out of routing"
- "Always use `@/` for absolute imports — never relative `../../`"
- "Adding a field to a Convex schema must use `v.optional()` for backward
  compatibility"
- "Server Components are the default — only add `'use client'` when the
  component needs state, effects, or browser APIs"

**8. Pre-PR Checks** (2–3 lines)

```bash
pnpm --filter @repo/web typecheck && \
pnpm --filter @repo/web lint && \
pnpm --filter @repo/web test && \
pnpm --filter @repo/web build
```

---

## Phase 4: Special Considerations

For each agents.md file, also consider:

### A. Design System / UI Package

```markdown
## Design System
- Components:    `src/components/ui/**`
- Tokens:        `src/styles/variables.css` (never hardcode colors or spacing)
- Naming:        All UI components are PascalCase — `Button.tsx`, `Input.tsx`
- File header:   Required on every component file
- Prop docs:     Required on every component (see Phase 0.3 component block)
- Examples:
  - Button:      Copy pattern from `src/components/ui/Button/Button.tsx`
  - Input:       Copy pattern from `src/components/ui/Input/Input.tsx`
  - Modal:       Copy pattern from `src/components/ui/Modal/Modal.tsx`
```

### B. Database / Data Layer (Convex)

```markdown
## Convex Data Layer
- Schema:        `convex/schema.ts` — ALL tables defined here
- All new fields: must use `v.optional()` for backward compatibility
- File header:   Required on every convex/*.ts file
- Function docs: Required on every exported query/mutation/action
- Naming:
  - Public functions:   camelCase — `getUserData`, `sendMessage`
  - Internal functions: camelCase — `processQueue`, `generateSummary`
  - Schema tables:      camelCase singular — `users`, `messages`, `products`
- Examples:
  - Query:       See `convex/users.ts → getMyProfile`
  - Mutation:    See `convex/messages/index.ts → sendMessage`
  - Action:      See `convex/messages/index.ts → generateAiReply`
```

### C. API / Backend Service

```markdown
## API Patterns
- Route handlers: `src/app/api/[domain]/route.ts` — always lowercase
- Naming:         Handler functions are named GET, POST, PUT, DELETE (uppercase)
- File header:    Required on every route.ts file
- Validation:     Zod schemas in `src/lib/validators/`
- Error handling: All errors thrown as typed ApiError
- Example:        See `src/app/api/users/route.ts` for the full pattern
```

### D. Testing

```markdown
## Testing
- Unit tests:        `*.test.ts` — colocated with the source file
- Integration tests: `tests/integration/**`
- E2E tests:         `tests/e2e/**` (Playwright)
- Naming:            Test files match the source file name exactly
                     `Button.tsx` → `Button.test.tsx`
                     `authService.ts` → `authService.test.ts`
- File header:       Required on every test file
- Run single test:   `pnpm test -- path/to/file.test.ts`
- Coverage target:   > 80%
```

---

## Output Format

Provide the files in this order:

1. **Phase 0 Confirmation** — Confirm the hybrid folder structure has been
   scaffolded and all naming conventions have been reviewed before proceeding
2. **Analysis Summary** (from Phase 1)
3. **Root agents.md** (complete, ready to copy)
4. **Each Sub-Folder agents.md** (one at a time, with file path shown)

For each file, use this format:

````
---
File: `agents.md` (root)
---
[full content here]

---
File: `apps/web/agents.md`
---
[full content here]

---
File: `convex/agents.md`
---
[full content here]
````

---

## Constraints & Quality Checks

Before generating any file, verify every item on this list:

### Structure Checks
- [ ] Hybrid folder structure scaffolded BEFORE any code was written
- [ ] All required global layer folders exist (`components/`, `hooks/`,
      `services/`, `utils/`, `types/`, `lib/`, `stores/`, `config/`)
- [ ] All route-level private folders use underscore prefix (`_components/`,
      `_hooks/`, `_lib/`)
- [ ] Route groups use parentheses (`(auth)/`, `(main)/`)
- [ ] Empty folders include a `.gitkeep` file

### Naming Convention Checks
- [ ] All folder names are lowercase and plural where appropriate
- [ ] All component files are PascalCase `.tsx`
- [ ] All non-component files are camelCase `.ts`
- [ ] All Next.js reserved files are lowercase (`page.tsx`, `layout.tsx`)
- [ ] All React component functions are PascalCase
- [ ] All non-component functions are camelCase with a verb-noun pattern
- [ ] All page default exports are named `[Segment]Page` (e.g., `DashboardPage`)
- [ ] All boolean variables start with `is`, `has`, `can`, or `should`
- [ ] All constants are SCREAMING_SNAKE_CASE
- [ ] All event handlers are prefixed with `handle`

### Documentation Checks
- [ ] Every file has a file header block at the top
- [ ] Every exported function has a function documentation block
- [ ] Every React component has a component documentation block with props listed
- [ ] Every custom hook has a hook documentation block
- [ ] All inline comments explain WHY, not what
- [ ] All TODO/FIXME comments include author, date, and ticket number

### agents.md Quality Checks
- [ ] Root agents.md is under 200 lines
- [ ] Root agents.md links to all sub-agents.md files
- [ ] Each sub-file has concrete examples with actual file paths
- [ ] Commands are copy-paste ready (no placeholders unless unavoidable)
- [ ] No duplication between root and sub-files
- [ ] JIT hints use `rg`, `find`, or glob patterns from the actual codebase
- [ ] Every `✅ DO` references a real file in the project
- [ ] Every `❌ DON'T` references a real anti-pattern or legacy file
- [ ] Pre-PR checks are single copy-paste commands

---

## Agent Behavior Rules (Always Active)

These rules apply to every action the agent takes throughout all phases:

```
1. NEVER create a file without applying the file header block first.

2. NEVER name a folder, file, function, variable, or component
   without verifying it against the naming conventions in Phase 0.2.

3. NEVER write a function without adding the documentation block above it.

4. NEVER place a component inside app/ without an underscore-prefixed
   private folder (_components/) unless it IS the route file itself.

5. NEVER use relative imports like ../../ — always use @/ absolute imports.

6. ALWAYS scaffold the complete folder structure before writing
   any business logic.

7. ALWAYS put the file header block as the very first content in
   every new file, before any imports.

8. ALWAYS use v.optional() when adding new fields to an existing
   Convex schema.

9. NEVER add "use client" unless the component explicitly needs
   useState, useEffect, or browser APIs.

10. ALWAYS check: does this belong in the global layer or in a
    feature-specific private folder before deciding where to create it.
```
