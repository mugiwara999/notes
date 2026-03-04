# Next.js Interview Notes

> Conceptual understanding + architectural decisions for web development interviews.

---

## Table of Contents

1. [Core Next.js Fundamentals](#1-core-nextjs-fundamentals)
2. [Rendering Strategies](#2-rendering-strategies)
3. [Server Components vs Client Components](#3-server-components-vs-client-components)
4. [Data Fetching](#4-data-fetching)
5. [API Routes / Backend](#5-api-routes--backend)
6. [Middleware](#6-middleware)
7. [Layouts and Templates](#7-layouts-and-templates)
8. [Navigation](#8-navigation)
9. [Performance Optimizations](#9-performance-optimizations)
10. [Authentication Patterns](#10-authentication-patterns)
11. [Environment Variables](#11-environment-variables)
12. [Deployment](#12-deployment)
13. [Error Handling](#13-error-handling)
14. [Caching](#14-caching)
15. [Streaming and Suspense](#15-streaming-and-suspense)
16. [SEO Features](#16-seo-features)
17. [Edge Runtime](#17-edge-runtime)
18. [Common Interview Questions](#18-common-interview-questions)
19. [Project-specific Questions](#19-project-specific-questions)

---

## 1. Core Next.js Fundamentals

### What Next.js Is

**Next.js** is a **React framework** that adds structure, tooling, and conventions on top of React. React alone is a **library** for building UIs; Next.js is a **full-stack framework** that decides how you structure files, fetch data, and render pages.

### What Problem Next.js Solves (vs Plain React)

| Aspect | React SPA (CRA, Vite) | Next.js |
|--------|------------------------|---------|
| **Rendering** | Client-side only (CSR) | SSR, SSG, ISR, CSR — you choose |
| **Routing** | Manual (React Router) | File-based, built-in |
| **SEO** | Poor (empty HTML until JS runs) | Strong (pre-rendered HTML) |
| **Backend** | Separate server needed | API routes in same project |
| **Build** | Single bundle, client downloads all | Code-split, server can render |

**Interview answer:** *"Next.js solves the problems of SEO, initial load performance, and full-stack development that you’d otherwise solve piecemeal with React + router + separate backend. It gives you a single framework with conventions for routing, data fetching, and rendering."*

### React SPA vs Next.js Framework

- **React SPA:** Browser downloads a big JS bundle, runs it, then fetches data and renders. First paint is slow; crawlers may see little content.
- **Next.js:** Server can render HTML per request (SSR) or at build time (SSG), so the user (and crawlers) get meaningful HTML immediately. JS then “hydrates” for interactivity.

### Benefits (Why Use Next.js)

1. **Server-Side Rendering (SSR)** — HTML generated on each request; good for dynamic, personalized, or frequently changing content.
2. **Static Site Generation (SSG)** — HTML generated at build time; fast, cacheable, good for blogs/docs.
3. **SEO** — Pre-rendered HTML means search engines and social previews get full content.
4. **File-based routing** — No manual route config; `app/blog/[slug]/page.tsx` → `/blog/hello-world`.
5. **Full-stack** — API routes, server components, middleware; one codebase for front + back.

---

### File-based Routing

#### App Router (`app/`) vs Pages Router (`pages/`)

- **App Router** (`app/`, Next.js 13+): Uses `layout.tsx`, `page.tsx`, `loading.tsx`, etc. Default and recommended.
- **Pages Router** (`pages/`): File name = route. `pages/about.tsx` → `/about`. Still supported but not the default.

We focus on **App Router** below.

#### Basic structure

```
app/
├── layout.tsx          → Root layout (wraps all pages)
├── page.tsx            → /
├── about/
│   └── page.tsx        → /about
├── blog/
│   └── [slug]/
│       └── page.tsx    → /blog/:slug (dynamic)
└── shop/
    └── [...slug]/
        └── page.tsx    → /shop/* (catch-all)
```

#### Dynamic routes

- **Single segment:** `[slug]`, `[id]` — one dynamic part.
- **Catch-all:** `[...slug]` — matches rest of path (e.g. `/shop/a/b/c` → `slug = ['a','b','c']`).
- **Optional catch-all:** `[[...slug]]` — same but also matches `/shop` (slug can be undefined).

**Example — dynamic page:**

```tsx
// app/blog/[slug]/page.tsx
export default async function BlogPost({
  params,
}: {
  params: Promise<{ slug: string }>;
}) {
  const { slug } = await params;
  const post = await getPost(slug);
  return <article>{post.title}</article>;
}
```

#### Route groups

Folders in parentheses don’t affect the URL: `(marketing)/about/page.tsx` → `/about`, not `/marketing/about`. Use for organizing without changing routes.

```
app/
├── (marketing)/
│   ├── about/page.tsx
│   └── blog/page.tsx
├── (dashboard)/
│   └── settings/page.tsx
```

#### Layouts

- **Layout:** Wraps child routes, keeps state (e.g. sidebar). Defined by `layout.tsx` in that segment.
- **Page:** Actual route UI. Defined by `page.tsx`.

---

## 2. Rendering Strategies

### Server-Side Rendering (SSR)

- **When:** Runs **on every request**.
- **Use for:** User-specific data, real-time content, pages that must never be stale (e.g. dashboard, cart).
- **How in App Router:** Use async Server Components and fetch on the server. To force no static optimization, use `fetch(..., { cache: 'no-store' })` or `dynamic = 'force-dynamic'`.

```tsx
// app/dashboard/page.tsx
export const dynamic = 'force-dynamic'; // opt out of static

export default async function Dashboard() {
  const data = await fetch('https://api.example.com/me', {
    cache: 'no-store',
  }).then((r) => r.json());
  return <div>Hello, {data.name}</div>;
}
```

**Interview:** *"SSR is used when the page content depends on the request (user, cookies, time) or must always be fresh. The server runs your component and fetch on each request and sends HTML."*

---

### Static Site Generation (SSG)

- **When:** Page is built **at build time** and reused for all users.
- **Use for:** Blogs, docs, marketing pages — content that doesn’t change per user or per request.
- **How:** Default in App Router when you don’t use dynamic APIs (cookies, headers, searchParams) and don’t set `dynamic` or `revalidate`. Fetch is cached.

```tsx
// app/blog/page.tsx — static by default
export default async function BlogList() {
  const posts = await fetch('https://api.example.com/posts').then((r) =>
    r.json()
  );
  return (
    <ul>
      {posts.map((p) => (
        <li key={p.id}>{p.title}</li>
      ))}
    </ul>
  );
}
```

---

### Incremental Static Regeneration (ISR)

- **When:** Page is static but can be **revalidated** after a time or on demand.
- **Use for:** Content that updates periodically (e.g. product list, blog index).

```tsx
// app/products/page.tsx
export const revalidate = 60; // revalidate at most every 60 seconds

export default async function Products() {
  const products = await fetch('https://api.example.com/products').then(
    (r) => r.json()
  );
  return <ProductList products={products} />;
}
```

- **Time-based:** `revalidate = 60` — after 60s, next request triggers background revalidation.
- **On-demand:** `revalidatePath('/products')` or `revalidateTag('products')` from a Route Handler or Server Action.

**Interview:** *"ISR lets you keep the performance of static pages but refresh content on an interval or when data changes, so you don’t need a full rebuild."*

---

### Client-Side Rendering (CSR)

- **When:** Data is fetched in the browser after the page loads (e.g. `useEffect` + `fetch` or React Query).
- **Use for:** Highly interactive UI, user-specific widgets, data that doesn’t affect SEO or first paint.

```tsx
'use client';

import { useEffect, useState } from 'react';

export default function LiveChart() {
  const [data, setData] = useState(null);
  useEffect(() => {
    fetch('/api/live-data').then((r) => r.json()).then(setData);
  }, []);
  return data ? <Chart data={data} /> : <Spinner />;
}
```

**When to use which (short):**

- **SSR:** Per-request, personalized, or always-fresh.
- **SSG:** Same for everyone, rarely changing.
- **ISR:** Same as SSG but with revalidation.
- **CSR:** After load, optional for SEO, highly interactive.

---

## 3. Server Components vs Client Components

### Default: Server Components (App Router)

In `app/`, every component is a **Server Component** unless you add `"use client"`. They run only on the server; their code is not sent to the client.

**Advantages:**

- **Smaller JS bundle** — Server Component code stays on server.
- **Direct data access** — DB, env, files; no extra API layer.
- **Security** — Secrets never sent to browser.
- **Performance** — No client JS for pure server components.

```tsx
// app/posts/page.tsx — Server Component (no directive)
import { db } from '@/lib/db';

export default async function PostsPage() {
  const posts = await db.post.findMany(); // direct DB access
  return (
    <ul>
      {posts.map((p) => (
        <li key={p.id}>{p.title}</li>
      ))}
    </ul>
  );
}
```

---

### Client Components

Add `"use client"` at the top when you need:

- **State:** `useState`, `useReducer`
- **Effects:** `useEffect`, `useLayoutEffect`
- **Event handlers:** `onClick`, `onChange`
- **Browser APIs:** `window`, `localStorage`, etc.
- **Client-only libraries** that use the above

```tsx
'use client';

import { useState } from 'react';

export default function Counter() {
  const [n, setN] = useState(0);
  return (
    <button onClick={() => setN((x) => x + 1)}>Count: {n}</button>
  );
}
```

**Interview — Why does Next.js default to Server Components?**  
*"To keep the client bundle small and to allow safe, direct server-side data access. You only send JS for parts that need interactivity. Most of the page can stay on the server."*

**Composition:** You can import Client Components inside Server Components and pass props (they must be serializable). Server Components cannot be imported inside Client Components; pass them as `children` or props instead.

```tsx
// Server Component
export default function Page() {
  return (
    <div>
      <ClientButton />  {/* OK */}
    </div>
  );
}
```

---

## 4. Data Fetching

### Fetching in Server Components

Use async/await. Fetch runs on the server; no `useEffect` needed.

```tsx
export default async function Page() {
  const res = await fetch('https://api.example.com/data');
  const data = await res.json();
  return <div>{data.title}</div>;
}
```

Next.js extends `fetch` with caching and revalidation (see Caching below).

### Fetch caching (Request Memoization + Data Cache)

- **Default:** `fetch` is cached (same URL in the same request is deduplicated; across requests it uses the Data Cache).
- **`cache: 'force-cache'`** (default) — use cache, cache on miss.
- **`cache: 'no-store'`** — no cache; always fresh (SSR-like).
- **`next: { revalidate: 60 }`** — revalidate after 60 seconds (ISR-like).

```tsx
// Always fresh (SSR)
const data = await fetch(url, { cache: 'no-store' });

// Revalidate every 60s (ISR)
const data = await fetch(url, { next: { revalidate: 60 } });
```

### Parallel vs sequential

- **Parallel:** Await multiple fetches together so they run in parallel.

```tsx
export default async function Page() {
  const [user, posts] = await Promise.all([
    fetch('/api/user').then((r) => r.json()),
    fetch('/api/posts').then((r) => r.json()),
  ]);
  return <Dashboard user={user} posts={posts} />;
}
```

- **Sequential:** One fetch depends on another.

```tsx
const user = await fetch('/api/user').then((r) => r.json());
const orders = await fetch(`/api/orders/${user.id}`).then((r) => r.json());
```

Use **parallel** when fetches are independent to reduce total time.

---

## 5. API Routes / Backend

Next.js can serve HTTP API endpoints via **Route Handlers** in the App Router.

### Location and methods

File: `app/api/users/route.ts` → endpoint: `GET/POST /api/users`.

Export named functions: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, etc.

```ts
// app/api/users/route.ts
import { NextResponse } from 'next/server';

export async function GET() {
  const users = await db.user.findMany();
  return NextResponse.json(users);
}

export async function POST(request: Request) {
  const body = await request.json();
  const user = await db.user.create({ data: body });
  return NextResponse.json(user, { status: 201 });
}
```

```ts
// app/api/users/[id]/route.ts
export async function GET(
  request: Request,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params;
  const user = await db.user.findUnique({ where: { id } });
  if (!user) return NextResponse.json(null, { status: 404 });
  return NextResponse.json(user);
}

export async function DELETE(
  request: Request,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params;
  await db.user.delete({ where: { id } });
  return new NextResponse(null, { status: 204 });
}
```

**When to use Route Handlers vs Express:**  
Use Route Handlers when the API is tightly coupled to the same Next.js app (same deploy, shared types, server actions calling them). Use Express (or another backend) when you need a separate service, different scaling, or complex middleware not covered by Next.js.

---

## 6. Middleware

**Runs before the request reaches the route** — same process as the app, on the Edge by default.

**Typical uses:**

- Authentication (check cookie/token, redirect to login).
- Redirects (locale, A/B, maintenance).
- Logging, headers, rewrite.

File must be at root: `middleware.ts` (or `src/middleware.ts`).

```ts
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const token = request.cookies.get('auth-token');
  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }
  return NextResponse.next();
}

export const config = {
  matcher: ['/dashboard/:path*', '/api/protected/:path*'],
};
```

**Interview:** *"Middleware runs before the request hits the page or API route. We use it to protect routes by checking auth and redirecting unauthenticated users, or to add headers and redirects."*

---

## 7. Layouts and Templates

### Layouts (`layout.tsx`)

- **Persistent** across navigations; state in layout is kept (e.g. sidebar open/closed).
- **Shared UI:** nav, sidebar, providers.
- **Nested:** Each segment can have its own `layout.tsx`; they nest (root → segment → segment).

```tsx
// app/layout.tsx
export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <nav>...</nav>
        {children}
        <footer>...</footer>
      </body>
    </html>
  );
}
```

```tsx
// app/dashboard/layout.tsx
export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div>
      <aside>Dashboard sidebar</aside>
      <main>{children}</main>
    </div>
  );
}
```

### Templates (`template.tsx`)

Like layout but **re-mounts on navigation** (new instance per route). Use when you need to reset state or run effects on every route change; otherwise prefer layout.

---

## 8. Navigation

### `<Link>`

Client-side navigation; prefetches visible links by default.

```tsx
import Link from 'next/link';

<Link href="/about">About</Link>
<Link href={`/blog/${post.slug}`}>Read more</Link>
<Link href="/dashboard" prefetch={false}>Dashboard</Link>
```

### `useRouter` (App Router)

From `next/navigation` (not `next/router` in App Router).

```tsx
'use client';

import { useRouter } from 'next/navigation';

export default function Form() {
  const router = useRouter();
  return (
    <button
      onClick={() => {
        router.push('/dashboard');   // add to history
        // router.replace('/dashboard');  // replace current
        // router.back();
        // router.refresh();  // refresh server components
      }}
    >
      Submit
    </button>
  );
}
```

---

## 9. Performance Optimizations

### Image: `next/image`

- Lazy loading, responsive `sizes`, automatic WebP/AVIF when supported.
- Prevents layout shift with width/height or fill.

```tsx
import Image from 'next/image';

<Image
  src="/hero.jpg"
  alt="Hero"
  width={800}
  height={400}
  priority
/>
<Image
  src={user.avatar}
  alt={user.name}
  width={40}
  height={40}
  className="rounded-full"
/>
```

### Fonts: `next/font`

- Self-hosted fonts; no layout shift; no extra network request to Google.

```tsx
import { Inter } from 'next/font/google';

const inter = Inter({ subsets: ['latin'] });

export default function RootLayout({ children }) {
  return (
    <html className={inter.className}>
      <body>{children}</body>
    </html>
  );
}
```

### Code splitting

- **Automatic per route** — each `page.tsx` and its tree are in separate chunks. Only load the JS for the current route.
- **Dynamic import:** For heavy client components, load them only when needed.

```tsx
import dynamic from 'next/dynamic';

const HeavyChart = dynamic(() => import('@/components/HeavyChart'), {
  loading: () => <p>Loading chart...</p>,
  ssr: false,
});
```

**Interview:** *"Next.js improves performance by code-splitting per route, optimizing images and fonts, and moving as much as possible to Server Components so the client bundle stays small."*

---

## 10. Authentication Patterns

Common patterns:

- **JWT:** Token in cookie (httpOnly) or sent in header; middleware or API checks it.
- **Sessions:** Session ID in cookie; server stores session (DB/Redis); middleware validates.
- **NextAuth.js:** Handles OAuth, credentials, sessions, callbacks; good for “sign in with Google” + custom logic.

**Middleware protection example:**

```ts
// middleware.ts
export function middleware(request: NextRequest) {
  const session = request.cookies.get('session');
  if (!session && request.nextUrl.pathname.startsWith('/app')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }
  return NextResponse.next();
}
```

**API route with auth:**

```ts
// app/api/protected/route.ts
export async function GET(request: Request) {
  const token = request.cookies.get('token')?.value;
  if (!token) return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  // verify JWT, load user, return data
  return NextResponse.json({ data: '...' });
}
```

**Interview:** *"We use middleware to redirect unauthenticated users from protected paths, and in API routes we read the cookie or Authorization header, verify the token/session, and return 401 if invalid."*

---

## 11. Environment Variables

- **File:** `.env.local` (git-ignored); also `.env`, `.env.development`, `.env.production`.
- **Server-only:** Any variable **without** `NEXT_PUBLIC_` is only available on the server (Node, Server Components, API routes, middleware).
- **Client-exposed:** `NEXT_PUBLIC_*` is inlined at build time and visible in the browser. Use only for non-secret, public config (e.g. public API URL).

```bash
# .env.local
DATABASE_URL=...           # server only
NEXT_PUBLIC_APP_URL=...    # available in browser
```

```ts
// Server
const dbUrl = process.env.DATABASE_URL;

// Client (only NEXT_PUBLIC_*)
const appUrl = process.env.NEXT_PUBLIC_APP_URL;
```

---

## 12. Deployment

### Vercel (native)

- Connect repo; Vercel runs `next build` and serves with their Edge/Node runtime.
- Preview deployments per branch/PR; env vars in dashboard.

### Self-hosting (Node)

```bash
npm run build   # next build
npm run start   # next start (production server)
```

Runs a Node server; put it behind a reverse proxy (Nginx, Caddy) and use PM2 or systemd.

### Docker

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build
EXPOSE 3000
CMD ["npm", "start"]
```

**Interview:** *"We can deploy to Vercel for zero-config and previews, or self-host with `next build` and `next start` behind a reverse proxy, or containerize with Docker for our own infra."*

---

## 13. Error Handling

Special files in the App Router:

| File | Purpose |
|------|--------|
| `error.tsx` | Error boundary for that segment and below; shows fallback and can log. |
| `loading.tsx` | Fallback UI while segment is loading (wraps in Suspense). |
| `not-found.tsx` | Shown when `notFound()` is called in that segment. |
| `global-error.tsx` | Root-level error boundary (must define own `<html>`). |

```tsx
// app/dashboard/error.tsx
'use client';

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <div>
      <p>{error.message}</p>
      <button onClick={reset}>Try again</button>
    </div>
  );
}
```

```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return <div>Loading dashboard...</div>;
}
```

```tsx
// app/not-found.tsx
export default function NotFound() {
  return <div>404 - Page not found</div>;
}
```

```tsx
// Trigger 404 from a Server Component or Route Handler
import { notFound } from 'next/navigation';

export default async function Page({ params }) {
  const post = await getPost(params.slug);
  if (!post) notFound();
  return <Article post={post} />;
}
```

---

## 14. Caching

Next.js has several layers:

### Request Memoization (per request)

- Deduplicates `fetch` and some other calls with the same arguments **within a single request**. No configuration.

### Data Cache (fetch)

- **Caching store** for `fetch` results across requests.
- Controlled by `cache` and `next.revalidate` on `fetch`, or by `revalidatePath` / `revalidateTag`.

### Full Route Cache (static pages)

- Output of rendering is cached when the page is static (no dynamic behavior).
- Opt out: `dynamic = 'force-dynamic'`, or use `cookies()`/`headers()`/`searchParams` in the segment.

### Router Cache (client)

- Client caches RSC payloads for visited segments; prefetched segments are also cached. Soft navigation uses this cache.
- `router.refresh()` can refresh server data.

### Revalidation

- **Time-based:** `revalidate = 60` or `fetch(..., { next: { revalidate: 60 } })`.
- **On-demand:** `revalidatePath('/path')` or `revalidateTag('tag')` from Server Action or Route Handler.

```ts
import { revalidatePath, revalidateTag } from 'next/cache';

// After mutating data
revalidatePath('/products');
revalidateTag('products');
```

```tsx
fetch(url, { next: { tags: ['products'] } });
```

**Interview:** *"We use the default fetch cache for static data, `no-store` or `revalidate` for fresh or ISR data, and on-demand revalidation with tags or paths after mutations so the UI stays consistent with the backend."*

---

## 15. Streaming and Suspense

**Streaming:** Server sends HTML in chunks so the browser can show parts of the page before everything is ready.

**Suspense:** You wrap async parts in `<Suspense fallback={...}>`; the fallback shows until the async child resolves.

```tsx
import { Suspense } from 'react';

export default function Page() {
  return (
    <div>
      <header>Header</header>
      <Suspense fallback={<div>Loading posts...</div>}>
        <AsyncPostList />
      </Suspense>
      <Suspense fallback={<div>Loading sidebar...</div>}>
        <AsyncSidebar />
      </Suspense>
    </div>
  );
}

async function AsyncPostList() {
  const posts = await fetch('/api/posts').then((r) => r.json());
  return <ul>{posts.map((p) => <li key={p.id}>{p.title}</li>)}</ul>;
}
```

**Interview:** *"Streaming with Suspense lets the shell render first and then stream in slow data, so the user sees progress instead of a single long wait."*

---

## 16. SEO Features

- **Pre-rendered HTML** (SSR/SSG) so crawlers see content.
- **Metadata API** for title, description, open graph, etc.

```tsx
// app/layout.tsx or app/blog/[slug]/page.tsx
export const metadata = {
  title: 'My App',
  description: '...',
  openGraph: {
    title: 'My App',
    description: '...',
    images: ['/og.png'],
  },
};

// Or dynamic
export async function generateMetadata({ params }) {
  const post = await getPost(params.slug);
  return {
    title: post.title,
    description: post.excerpt,
  };
}
```

---

## 17. Edge Runtime

- **Node.js runtime (default):** Full Node APIs; runs in Node on Vercel or your server.
- **Edge runtime:** Lightweight, runs at the edge (close to user); no full Node APIs; lower latency for simple logic.

```ts
// app/api/hello/route.ts
export const runtime = 'edge';

export async function GET() {
  return Response.json({ hello: 'world' });
}
```

**Use Edge for:** Auth checks, redirects, A/B, simple APIs.  
**Use Node for:** Heavy deps, file system, long-running, DB drivers that don’t support Edge.

**Interview:** *"Edge runs at the edge with a smaller runtime and no full Node; it’s good for middleware-like logic and simple APIs. We use Node when we need full compatibility or heavier server work."*

---

## 18. Common Interview Questions

**Q: Why Next.js over plain React?**  
Next.js gives you file-based routing, SSR/SSG/ISR, API routes, and strong defaults for performance and SEO. With React alone you’d add a router, a backend, and your own data-fetching and build setup.

**Q: SSR vs SSG?**  
SSR runs on every request and is good for personalized or always-fresh content. SSG runs at build time and is good for static content; it’s faster and cheaper to serve. ISR is SSG with revalidation so content can update without full rebuilds.

**Q: Server vs Client components?**  
Server components run on the server only and don’t ship their code to the client; they’re default for smaller bundles and direct data access. Client components run in the browser for state, effects, and event handlers; we use them only where needed.

**Q: How does Next.js improve performance?**  
Code splitting per route, Server Components reducing client JS, image and font optimization, and caching (fetch cache, ISR, static generation) so we send less and render faster.

**Q: How does routing work?**  
App Router uses the file system: `app/` folders map to segments, `page.tsx` is the route UI, `layout.tsx` wraps segments. Dynamic segments use `[param]`; catch-all uses `[...slug]`. The framework matches the URL to the file tree and renders the corresponding layout + page.

**Q: How did you implement authentication?**  
We use [cookies/sessions/JWT] and middleware to protect routes: middleware checks the auth cookie and redirects to login for protected paths. API routes validate the same token/session and return 401 when invalid. [If you used NextAuth: we use NextAuth for OAuth and session management, with middleware to protect routes.]

---

## 19. Project-specific Questions

Prepare short, concrete answers from your own projects.

- **Why did you use Next.js?**  
  e.g. “We needed good SEO and fast first load, so we used SSG for marketing and blog and SSR for the dashboard. Having API routes in the same repo simplified deployment and sharing types.”

- **Why SSR/CSR/SSG for this page?**  
  e.g. “Product list is ISR so it’s fast and still updated every hour; cart is client-side because it’s highly interactive and doesn’t need to be in the initial HTML.”

- **How did you fetch data?**  
  e.g. “We fetch in Server Components with `fetch` and use `revalidate` for product pages; for real-time we use client-side polling or WebSockets in a Client Component.”

- **How did you handle authentication?**  
  e.g. “We use JWT in httpOnly cookies; middleware checks the cookie and redirects to login for `/app/*`; API routes verify the JWT before returning data.”

- **How did you optimize performance?**  
  e.g. “We use `next/image` for images, `next/font` for fonts, and keep most of the product page as Server Components; we only use Client Components for the cart and filters. We also use ISR with revalidate for the product listing.”

---

*End of Next.js interview notes.*
