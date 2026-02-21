# 📐 Next.js 16 — Project Rules & Conventions
# App Router | Hybrid Folder Structure (Layer + Feature Based)
# Version: 1.0.0
# Last Updated: 2026-02-21

---

## 📑 Table of Contents

1. [Philosophy & Overview](#philosophy--overview)
2. [Folder Structure](#folder-structure)
3. [App Router Special Files](#app-router-special-files)
4. [Folder Naming Rules](#folder-naming-rules)
5. [File Naming Rules](#file-naming-rules)
6. [Component Naming Rules](#component-naming-rules)
7. [Variable Naming Rules](#variable-naming-rules)
8. [Function Naming Rules](#function-naming-rules)
9. [TypeScript & Type Naming Rules](#typescript--type-naming-rules)
10. [Code Documentation & Comments](#code-documentation--comments)
11. [Next.js Specific Rules](#nextjs-specific-rules)
12. [Import Rules](#import-rules)
13. [Testing Rules](#testing-rules)
14. [Environment & Config Rules](#environment--config-rules)

---

## 1. Philosophy & Overview

This project uses a **Hybrid Folder Structure** that combines:

- **Layer-Based Architecture** — Global shared layers like `components/`, `hooks/`,
  `services/`, `utils/`, `types/` live at the `src/` level and are available across the
  entire application.
- **Feature-Based Architecture** — Each feature/domain (e.g. `auth`, `dashboard`,
  `products`) is a self-contained module that encapsulates its own components, hooks,
  services, and utilities, co-located close to its route.

> **Rule:** If a piece of code is used by **2 or more features**, promote it to
> the global shared layer. If it is used by **only 1 feature**, keep it inside
> that feature's folder.

---

## 2. Folder Structure

```

my-next-app/
├── public/ # Static assets (images, fonts, svgs, robots.txt)
│ ├── assets/
│ │ ├── fonts/
│ │ ├── images/
│ │ └── svgs/
│ ├── favicon.ico
│ └── robots.txt
│
├── src/
│ │
│ ├── app/ # ─── Next.js App Router (Routing Layer) ───
│ │ ├── layout.tsx # Root layout
│ │ ├── page.tsx # Home page ( / )
│ │ ├── loading.tsx # Global loading state
│ │ ├── error.tsx # Global error boundary
│ │ ├── not-found.tsx # Global 404 page
│ │ ├── globals.css # Global styles entry
│ │ │
│ │ ├── (auth)/ # Route Group: Public/Auth routes (no URL segment)
│ │ │ ├── layout.tsx # Auth-specific layout
│ │ │ ├── login/
│ │ │ │ ├── page.tsx
│ │ │ │ └── loading.tsx
│ │ │ └── register/
│ │ │ └── page.tsx
│ │ │
│ │ ├── (main)/ # Route Group: Protected/Authenticated routes
│ │ │ ├── layout.tsx # Main app layout (with sidebar/nav)
│ │ │ │
│ │ │ ├── dashboard/ # ─── Feature: Dashboard ───
│ │ │ │ ├── page.tsx
│ │ │ │ ├── layout.tsx
│ │ │ │ ├── loading.tsx
│ │ │ │ ├── error.tsx
│ │ │ │ ├── _components/ # Private: Dashboard-specific components
│ │ │ │ │ ├── DashboardChart.tsx
│ │ │ │ │ └── StatsCard.tsx
│ │ │ │ ├── _hooks/ # Private: Dashboard-specific hooks
│ │ │ │ │ └── useDashboardStats.ts
│ │ │ │ └── _lib/ # Private: Dashboard loaders/actions/services
│ │ │ │ ├── dashboard.actions.ts
│ │ │ │ └── dashboard.loader.ts
│ │ │ │
│ │ │ ├── products/ # ─── Feature: Products ───
│ │ │ │ ├── page.tsx
│ │ │ │ ├── layout.tsx
│ │ │ │ ├── loading.tsx
│ │ │ │ ├── error.tsx
│ │ │ │ ├── [productId]/ # Dynamic route
│ │ │ │ │ ├── page.tsx
│ │ │ │ │ └── loading.tsx
│ │ │ │ ├── _components/
│ │ │ │ │ ├── ProductCard.tsx
│ │ │ │ │ └── ProductList.tsx
│ │ │ │ ├── _hooks/
│ │ │ │ │ └── useProducts.ts
│ │ │ │ └── _lib/
│ │ │ │ ├── products.actions.ts
│ │ │ │ └── products.loader.ts
│ │ │ │
│ │ │ └── settings/ # ─── Feature: Settings ───
│ │ │ ├── page.tsx
│ │ │ └── _components/
│ │ │ └── SettingsForm.tsx
│ │ │
│ │ └── api/ # API Route Handlers
│ │ ├── auth/
│ │ │ └── [...nextauth]/
│ │ │ └── route.ts
│ │ └── products/
│ │ └── route.ts
│ │
│ ├── components/ # ─── Global Shared Components (Layer) ───
│ │ ├── ui/ # Atomic/primitive UI components
│ │ │ ├── Button/
│ │ │ │ ├── Button.tsx
│ │ │ │ ├── Button.test.tsx
│ │ │ │ └── index.ts
│ │ │ ├── Input/
│ │ │ │ ├── Input.tsx
│ │ │ │ └── index.ts
│ │ │ ├── Modal/
│ │ │ └── Card/
│ │ ├── layout/ # Layout-level shared components
│ │ │ ├── Header.tsx
│ │ │ ├── Footer.tsx
│ │ │ ├── Sidebar.tsx
│ │ │ └── Navbar.tsx
│ │ └── shared/ # Cross-feature shared components
│ │ ├── LoadingSpinner.tsx
│ │ ├── ErrorBoundary.tsx
│ │ └── PageWrapper.tsx
│ │
│ ├── hooks/ # ─── Global Custom Hooks (Layer) ───
│ │ ├── useDebounce.ts
│ │ ├── useLocalStorage.ts
│ │ ├── useMediaQuery.ts
│ │ └── useWindowSize.ts
│ │
│ ├── services/ # ─── Global Business Logic / API Clients (Layer) ───
│ │ ├── authService.ts
│ │ ├── productService.ts
│ │ └── userService.ts
│ │
│ ├── lib/ # ─── Third-Party Library Wrappers / Config (Layer) ───
│ │ ├── apiClient.ts # Axios / fetch wrapper
│ │ ├── auth.ts # NextAuth config
│ │ ├── db.ts # DB client (Prisma/Drizzle)
│ │ └── queryClient.ts # React Query client
│ │
│ ├── utils/ # ─── Pure Utility / Helper Functions (Layer) ───
│ │ ├── formatDate.ts
│ │ ├── formatCurrency.ts
│ │ ├── validations.ts
│ │ └── helpers.ts
│ │
│ ├── types/ # ─── Global TypeScript Types & Interfaces (Layer) ───
│ │ ├── api.types.ts
│ │ ├── auth.types.ts
│ │ ├── product.types.ts
│ │ └── index.ts
│ │
│ ├── stores/ # ─── Global State Management (Layer) ───
│ │ ├── authStore.ts
│ │ └── cartStore.ts
│ │
│ ├── styles/ # ─── Global Styles (Layer) ───
│ │ ├── variables.css
│ │ └── themes.css
│ │
│ └── config/ # ─── App-Level Configuration ───
│ ├── constants.ts
│ ├── routes.ts
│ └── siteConfig.ts
│
├── .env.local # Local environment variables (never commit)
├── .env.example # Example env file (safe to commit)
├── next.config.ts # Next.js configuration
├── tsconfig.json # TypeScript configuration
├── tailwind.config.ts # Tailwind CSS config
├── eslint.config.mjs # ESLint 9 config
├── prettier.config.js # Prettier config
└── package.json

````

---

## 3. App Router Special Files

These are **Next.js 16 reserved file conventions** inside the `app/` directory.
Always use them at the appropriate route segment level.

| File              | Purpose                                              |
|-------------------|------------------------------------------------------|
| `layout.tsx`      | Shared UI that wraps child segments (persists across navigation) |
| `page.tsx`        | The public-facing UI for a route segment             |
| `loading.tsx`     | Suspense-based loading skeleton/spinner UI           |
| `error.tsx`       | Error boundary for the route segment                 |
| `not-found.tsx`   | 404 UI for the route segment                         |
| `template.tsx`    | Like layout, but re-mounts on navigation             |
| `route.ts`        | API Route Handler (replaces `pages/api`)             |
| `middleware.ts`   | Edge middleware (placed at `src/` root)              |
| `sitemap.ts`      | Generates sitemap.xml                                |
| `robots.ts`       | Generates robots.txt                                 |
| `opengraph-image` | OG image generation                                  |
| `default.tsx`     | Fallback UI for parallel routes                      |

### Route Segment Conventions

| Convention           | Example                    | Behavior                                   |
|----------------------|----------------------------|--------------------------------------------|
| Static folder        | `dashboard/`               | Maps to `/dashboard` URL                   |
| Route Group          | `(auth)/`                  | Groups routes without affecting URL        |
| Dynamic Segment      | `[productId]/`             | Matches any value: `/products/123`         |
| Catch-All Segment    | `[...slug]/`               | Matches all paths: `/docs/a/b/c`           |
| Optional Catch-All   | `[[...slug]]/`             | Matches with or without params             |
| Private Folder       | `_components/`             | Excluded from routing — for co-location    |
| Parallel Route Slot  | `@analytics/`              | Named slot rendered in the parent layout   |
| Intercepting Route   | `(.)login/`                | Intercepts a route and shows it as a modal |

---

## 4. Folder Naming Rules

### ✅ Use lowercase only

```bash
# ✅ Good
components/
pages/
services/
utils/
hooks/
auth/
dashboard/
products/

# ❌ Bad
Components/
PagesFolder/
MyServices/
Dashboard_Feature/
````

### ✅ Use meaningful, descriptive names

```bash
# ✅ Good
auth/
dashboard/
products/
shared/
settings/

# ❌ Bad
folder1/
test23/
newFolder/
stuff/
misc/
```

### ✅ Use plural names for collections

```bash
# ✅ Good
components/
services/
hooks/
utils/
types/
stores/

# ❌ Bad
component/
service/
hook/
util/
type/
```

### ✅ Use underscore prefix for private/co-located folders inside `app/`

```bash
# ✅ Good — excluded from routing
_components/
_hooks/
_lib/

# ❌ Bad — accidentally creates a URL route
components/   ← inside app/ without underscore
```

### ✅ Use parentheses for Route Groups (no URL impact)

```bash
# ✅ Good
(auth)/
(main)/
(marketing)/

# ❌ Bad — route groups should use parentheses, not underscores or prefixes
auth-group/
_auth/
```

---

## 5. File Naming Rules

### ✅ Component and Page files → PascalCase `.tsx`

```bash
# ✅ Good
Header.tsx
LoginForm.tsx
UserProfile.tsx
ProductCard.tsx
DashboardChart.tsx

# ❌ Bad
header.tsx
login_form.tsx
userprofile.tsx
dashboard-chart.tsx
```

### ✅ Non-component files (services, utils, hooks, lib, types) → camelCase `.ts`

```bash
# ✅ Good
authService.ts
apiClient.ts
validations.ts
formatDate.ts
useDebounce.ts
useDashboardStats.ts
productService.ts
helpers.ts

# ❌ Bad
AuthService.ts
AUTH.ts
Validation.ts
USE-DEBOUNCE.ts
```

### ✅ Next.js App Router reserved files → lowercase `.tsx` or `.ts`

```bash
# ✅ Correct (Next.js convention — must be lowercase)
page.tsx
layout.tsx
loading.tsx
error.tsx
not-found.tsx
route.ts
template.tsx
middleware.ts
```

### ✅ Test files → same name as the file being tested + `.test.ts` / `.test.tsx`

```bash
# ✅ Good
Button.test.tsx
authService.test.ts
formatDate.test.ts
```

### ✅ Index barrel files → lowercase `index.ts`

```bash
# ✅ Good
components/ui/Button/index.ts
types/index.ts
```

---

## 6. Component Naming Rules

### ✅ All React components → PascalCase

```tsx
// ✅ Good
function LoginForm() {}
function UserCard() {}
function DashboardChart() {}
const ProductList = () => {};

// ❌ Bad
function user_card() {}
function loginform() {}
const productlist = () => {};
```

### ✅ Server Components vs Client Components

- By default in Next.js 16 App Router, all components are **Server Components**.
- Add `"use client"` directive **only** when the component needs:
  - React state (`useState`)
  - React effects (`useEffect`)
  - Browser APIs
  - Event listeners

```tsx
// ✅ Client Component — only when needed
"use client";

function LoginForm() {
  const [email, setEmail] = useState("");
  // ...
}

// ✅ Server Component — default, no directive needed
async function ProductList() {
  const products = await fetchProducts();
  return <ul>{products.map(...)}</ul>;
}
```

### ✅ Suffix components by their role for clarity

```tsx
// ✅ Good naming by role
UserCard.tsx       // display card
LoginForm.tsx      // form component
AuthProvider.tsx   // context provider
useAuthStore.ts    // zustand store hook
ProductService.ts  // service class (if OOP)
```

---

## 7. Variable Naming Rules

### ✅ Use camelCase for all variables

```ts
// ✅ Good
let userName;
let totalPrice;
let currentUserId;
let productCount;

// ❌ Bad
let Total_Price;
let login;
let USERNAME;
let current_user_id;
```

### ✅ Boolean variables must start with `is`, `has`, `can`, or `should`

```ts
// ✅ Good
let isLoading: boolean;
let hasError: boolean;
let canEdit: boolean;
let shouldRedirect: boolean;
let isAuthenticated: boolean;

// ❌ Bad
let loading: boolean;
let error: boolean;
let edit: boolean;
```

### ✅ Constants → SCREAMING_SNAKE_CASE

```ts
// ✅ Good
const MAX_RETRY_ATTEMPTS = 3;
const API_BASE_URL = "https://api.example.com";
const DEFAULT_PAGE_SIZE = 20;

// ❌ Bad
const maxRetryAttempts = 3;
const apibaseurl = "https://api.example.com";
```

### ✅ Arrays should have plural names

```ts
// ✅ Good
const products = [];
const userIds = [];
const menuItems = [];

// ❌ Bad
const productList = [];
const dataArr = [];
```

---

## 8. Function Naming Rules

### ✅ Use camelCase for all functions

```ts
// ✅ Good
function getUserData() {}
function fetchProducts() {}
function calculateTotalPrice() {}
function handleFormSubmit() {}

// ❌ Bad
function GetUserData() {}  // don't use PascalCase for non-component functions
function do_task() {}
function doTask() {}       // too vague
function calc() {}         // too abbreviated
```

### ✅ Use meaningful, descriptive verb-noun pattern

```ts
// ✅ Good — verb + noun describes the action
getUserById()
fetchAllProducts()
calculateTotalPrice()
validateEmailAddress()
formatDateToISO()
handleLoginSubmit()
transformApiResponse()

// ❌ Bad — too vague
doTask()
process()
run()
handleData()
doStuff()
```

### ✅ Event handler functions → prefix with `handle`

```tsx
// ✅ Good
function handleButtonClick() {}
function handleFormSubmit() {}
function handleInputChange() {}

// ❌ Bad
function buttonClick() {}
function submit() {}
function onChange() {}
```

### ✅ Data-fetching functions → prefix with `fetch` or `get`

```ts
// ✅ Good — async server fetching
async function fetchUserById(userId: string) {}
async function fetchAllProducts() {}

// ✅ Good — sync getter
function getUserFromCache(userId: string) {}
```

### ✅ Server Actions → use descriptive past-tense or action-noun form

```ts
// ✅ Good (in .actions.ts files)
async function createProduct(formData: FormData) {}
async function deleteProductById(id: string) {}
async function updateUserProfile(data: UpdateUserInput) {}
```

---

## 9. TypeScript & Type Naming Rules

### ✅ Interfaces → PascalCase prefixed with `I` (optional but consistent)

```ts
// ✅ Good (team preference — pick one and stick to it)
interface User {
  id: string;
  name: string;
  email: string;
}

// Or with I prefix (also acceptable if your team prefers it)
interface IUser {
  id: string;
  name: string;
}
```

### ✅ Type Aliases → PascalCase

```ts
// ✅ Good
type ProductStatus = "active" | "inactive" | "draft";
type ApiResponse<T> = {
  data: T;
  message: string;
  success: boolean;
};
```

### ✅ Enums → PascalCase name, SCREAMING_SNAKE_CASE values

```ts
// ✅ Good
enum UserRole {
  ADMIN = "ADMIN",
  USER = "USER",
  MODERATOR = "MODERATOR",
}
```

### ✅ Generic type parameters → single uppercase letter or descriptive name

```ts
// ✅ Good
function fetchData<T>(url: string): Promise<T> {}
type ApiResponse<TData> = { data: TData };
```

---

## 10. Code Documentation & Comments

### ✅ File Header Block — Required for all files

Every file must begin with this header block comment:

```ts
/*
 * File Name:     authService.ts
 * Description:   This file handles all authentication-related API calls,
 *                including login, logout, token refresh, and session validation.
 * Author:        Yasir Khan
 * Created Date:  2026-02-21
 * Last Modified: 2026-02-21
 */
```

### ✅ Function / Component Documentation Block

All exported functions and React components must have a JSDoc-style block.

#### For a regular function:

```ts
/*
 * Function Name: getUserData
 * Description:   Fetches user data from the API by user ID.
 * Parameters:    userId (string) — The unique identifier of the user.
 * Returns:       Promise<User> — A promise resolving to the User object.
 */
export async function getUserData(userId: string): Promise<User> {
  // implementation
}
```

#### For a React component (describe props):

```tsx
/*
 * Component Name: UserCard
 * Description:    Displays a user's profile card with their avatar,
 *                 name, and role badge.
 * Props:
 *   - user       (User)     — The user object to display.
 *   - isLoading  (boolean)  — Shows a skeleton if true.
 *   - onEdit     (function) — Callback triggered when Edit button is clicked.
 */
export function UserCard({ user, isLoading, onEdit }: UserCardProps) {
  // implementation
}
```

#### For a custom hook:

```ts
/*
 * Hook Name:   useDebounce
 * Description: Debounces a value by the specified delay. Useful for
 *              preventing excessive API calls on input changes.
 * Parameters:  value (T)       — The value to debounce.
 *              delay (number)  — Delay in milliseconds.
 * Returns:     T — The debounced value.
 */
export function useDebounce<T>(value: T, delay: number): T {
  // implementation
}
```

### ✅ Inline Comments — Explain "why", not "what"

```ts
// ✅ Good — explains the reasoning
// We delay by 300ms to avoid rate-limiting on the search API
const debouncedSearch = useDebounce(searchQuery, 300);

// ❌ Bad — describes the obvious
// Set delay to 300
const delay = 300;
```

### ✅ TODO and FIXME comments — always include author and date

```ts
// TODO [Yasir Khan, 2026-02-21]: Replace with React Query once API is ready.
// FIXME [Yasir Khan, 2026-02-21]: This breaks on Safari — investigate later.
```

---

## 11. Next.js Specific Rules

### ✅ Prefer Server Components by default

- All components inside `app/` are Server Components by default.
- Only add `"use client"` when the component needs interactivity.
- Keep `"use client"` boundaries as **low** in the tree as possible.

```tsx
// ✅ Good — push "use client" down to the leaf
// app/dashboard/page.tsx (Server Component)
import { DashboardChart } from "./_components/DashboardChart";
export default async function DashboardPage() {
  const data = await fetchDashboardData();
  return <DashboardChart data={data} />;
}

// app/dashboard/_components/DashboardChart.tsx (Client Component)
"use client";
export function DashboardChart({ data }) {
  // uses useState, recharts, etc.
}
```

### ✅ Server Actions go in `*.actions.ts` files — keep them thin

```ts
// ✅ Good — action is thin; business logic lives in service
"use server";
export async function createProductAction(formData: FormData) {
  const data = validateProductInput(formData); // validation util
  return await productService.createProduct(data); // business logic in service
}

// ❌ Bad — business logic crammed into the action
"use server";
export async function createProductAction(formData: FormData) {
  const name = formData.get("name");
  const price = Number(formData.get("price"));
  if (!name || price <= 0) throw new Error("Invalid");
  await db.product.create({ data: { name, price } }); // too much logic here
}
```

### ✅ Data loaders go in `*.loader.ts` files

```ts
// app/products/_lib/products.loader.ts
export async function loadProducts(page: number) {
  return await productService.getAll({ page });
}
```

### ✅ Use Route Groups to separate authenticated and public routes

```
app/
├── (auth)/      # Public routes: /login, /register
│   └── layout.tsx
└── (main)/      # Protected routes: /dashboard, /settings
    └── layout.tsx
```

### ✅ Use dynamic metadata exports for SEO

```tsx
// app/products/[productId]/page.tsx
export async function generateMetadata({ params }: Props) {
  const product = await fetchProductById(params.productId);
  return {
    title: `${product.name} | MyShop`,
    description: product.description,
  };
}
```

### ✅ Use `loading.tsx` for every major route segment

```tsx
// app/dashboard/loading.tsx
export default function DashboardLoading() {
  return <DashboardSkeleton />;
}
```

### ✅ Use `error.tsx` for every major route segment

```tsx
// app/dashboard/error.tsx
"use client"; // error boundaries must be Client Components
export default function DashboardError({ error, reset }) {
  return (
    <div>
      <p>Something went wrong: {error.message}</p>
      <button onClick={reset}>Try again</button>
    </div>
  );
}
```

---

## 12. Import Rules

### ✅ Use absolute imports with path aliases (configured in `tsconfig.json`)

```ts
// ✅ Good
import { Button } from "@/components/ui/Button";
import { authService } from "@/services/authService";
import { formatDate } from "@/utils/formatDate";
import type { User } from "@/types";

// ❌ Bad
import { Button } from "../../../components/ui/Button";
import { authService } from "../../services/authService";
```

### ✅ Import order (enforced by ESLint `import/order`)

```ts
// 1. Node built-ins
import path from "path";

// 2. External packages
import { z } from "zod";
import { useQuery } from "@tanstack/react-query";

// 3. Internal absolute imports
import { Button } from "@/components/ui/Button";
import { authService } from "@/services/authService";

// 4. Relative imports
import { DashboardChart } from "./_components/DashboardChart";

// 5. Type imports (last)
import type { User } from "@/types";
```

---

## 13. Testing Rules

### ✅ Test file naming

```bash
# Unit tests — co-located with the file
Button.test.tsx
authService.test.ts
formatDate.test.ts

# Integration / E2E tests
tests/
└── e2e/
    └── auth.spec.ts
```

### ✅ Test function naming — descriptive `it` or `describe` blocks

```ts
// ✅ Good
describe("authService", () => {
  it("should return a user when valid credentials are provided", async () => {});
  it("should throw an error when password is incorrect", async () => {});
});

// ❌ Bad
describe("authService", () => {
  it("test 1", async () => {});
  it("works", async () => {});
});
```

---

## 14. Environment & Config Rules

### ✅ Environment variable naming → `NEXT_PUBLIC_` prefix for client-side vars

```bash
# .env.local

# ✅ Public (accessible in browser)
NEXT_PUBLIC_APP_URL=https://myapp.com
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_live_...

# ✅ Private (server-only)
DATABASE_URL=postgresql://...
NEXTAUTH_SECRET=super-secret-value
STRIPE_SECRET_KEY=sk_live_...
```

### ✅ Always validate environment variables at startup

```ts
// src/config/env.ts
import { z } from "zod";

const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  NEXTAUTH_SECRET: z.string().min(1),
  NEXT_PUBLIC_APP_URL: z.string().url(),
});

export const env = envSchema.parse(process.env);
```

### ✅ Never commit `.env.local` — always maintain `.env.example`

```bash
# .env.example (safe to commit — no real values)
DATABASE_URL=
NEXTAUTH_SECRET=
NEXT_PUBLIC_APP_URL=
```

---

## 📌 Quick Reference Cheat Sheet

| What                       | Convention              | Example                        |
|----------------------------|-------------------------|--------------------------------|
| Folder names               | lowercase, plural       | `components/`, `hooks/`        |
| Feature folders            | lowercase, meaningful   | `auth/`, `dashboard/`          |
| Private app folders        | underscore prefix       | `_components/`, `_lib/`        |
| Route groups               | parentheses             | `(auth)/`, `(main)/`           |
| Component files            | PascalCase `.tsx`       | `UserCard.tsx`                 |
| Non-component files        | camelCase `.ts`         | `authService.ts`               |
| Next.js reserved files     | lowercase               | `page.tsx`, `layout.tsx`       |
| React components           | PascalCase function     | `function UserCard() {}`       |
| Variables                  | camelCase               | `let userName`                 |
| Boolean variables          | `is/has/can/should`     | `isLoading`, `hasError`        |
| Constants                  | SCREAMING_SNAKE_CASE    | `MAX_RETRY_ATTEMPTS`           |
| Functions                  | camelCase, verb-noun    | `getUserData()`                |
| Event handlers             | `handle` prefix         | `handleFormSubmit()`           |
| Types / Interfaces         | PascalCase              | `type ProductStatus`           |
| Enums                      | PascalCase + SCREAMING  | `enum UserRole { ADMIN }`      |
| File header comment        | Required on all files   | See documentation section      |
| Function doc comment       | Required on all exports | See documentation section      |
