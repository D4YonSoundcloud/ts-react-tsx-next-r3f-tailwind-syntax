# Next.js Syntax Reference

A single-page lookup for Next.js 16 with TypeScript: file conventions, the App Router, data fetching and caching, Server Functions, route handlers, and the type helpers Next generates for you.

This page assumes the [TypeScript](./typescript.md) and [React](./react.md) references. Server Components, `use`, Actions and `useActionState` are explained there; this page covers what Next adds on top.

**How to use this page.** One long file, so browser find (`Ctrl+F` / `Cmd+F`) is the search tool. Search for the file name or API — `generateStaticParams`, `proxy.ts`, `"use cache"`, `PageProps` — rather than a description.

| Mark | Meaning |
|---|---|
| `// →` | the value the expression produces at runtime |
| `// ^?` | the type TypeScript infers |
| `// ✗` | a compile or runtime error, followed by the message |
| `// ≤15` | how this was written before Next.js 16 |
| `// pages/` | the equivalent in the old Pages Router |

Checked against **Next.js 16.3**, **React 19.3**, **@types/react 19.3** and **TypeScript** under `strict: true`. Route types are the ones actually emitted by `next typegen`.

---

## Contents

[Setup](#setup) · [File conventions](#file-conventions) · [Pages](#pages) · [Layouts](#layouts) · [Dynamic segments](#dynamic-segments) · [Route groups](#route-groups) · [Parallel and intercepting routes](#parallel-and-intercepting-routes) · [Typed routes](#typed-routes) · [Link](#link) · [Navigation hooks](#navigation-hooks) · [Server vs client](#server-vs-client) · [Request APIs](#request-apis) · [Data fetching](#data-fetching) · [use cache](#use-cache) · [Revalidation](#revalidation) · [Server Functions](#server-functions) · [Forms](#forms) · [Route handlers](#route-handlers) · [proxy.ts](#proxyts) · [Metadata](#metadata) · [generateStaticParams](#generatestaticparams) · [Streaming](#streaming) · [Errors](#errors) · [redirect and notFound](#redirect-and-notfound) · [Images](#images) · [Fonts](#fonts) · [Scripts](#scripts) · [Environment variables](#environment-variables) · [next.config.ts](#nextconfigts) · [Idioms](#general-idioms) · [Gotchas](#gotchas) · [Next 15 → 16](#next-15--16) · [Versions](#version-notes)

---

## Setup

```bash
npx create-next-app@latest --typescript
npx next upgrade          # 16.1+: updates next, react and types together
```

```jsonc
// tsconfig.json — what create-next-app generates, annotated
{
  "compilerOptions": {
    "target": "es2022",
    "lib": ["dom", "dom.iterable", "es2023"],
    "module": "esnext",
    "moduleResolution": "bundler",

    "jsx": "preserve",           // Next runs the JSX transform itself
    "strict": true,
    "noEmit": true,              // Next compiles; tsc only type-checks
    "allowJs": true,
    "esModuleInterop": true,
    "isolatedModules": true,     // required — Next compiles file by file
    "incremental": true,
    "resolveJsonModule": true,
    "skipLibCheck": true,

    "paths": { "@/*": ["./*"] }, // the @ alias, used throughout this page
    "plugins": [{ "name": "next" }]   // editor-only: server/client boundary hints
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

Two `include` entries matter. `next-env.d.ts` declares the image and CSS-module modules; `.next/types/**/*.ts` is where the generated route types live, and without it `PageProps` and friends are undefined.

```bash
next typegen        # regenerate route types without a full build
next dev            # generates them on the fly
next build          # runs typegen, then type-checks the whole project
```

Never edit `next-env.d.ts` — it is regenerated. Put your own ambient declarations in a separate `.d.ts`.

```ts
// env.d.ts — your own additions (a global .d.ts, no imports/exports)
// not standalone-compilable
declare module "*.md?raw" {
  const content: string;
  export default content;
}
```

Note that `next-env.d.ts` already declares `*.svg`, `*.png` and the CSS-module
patterns. Re-declaring one of those in your own file is a duplicate-identifier
error, not an override — use a distinct specifier, or a custom loader suffix.

---

## File conventions

Everything in `app/` is routed by folder name. Only these file names are special.

| File | Purpose | May be async |
|---|---|---|
| `page.tsx` | a route's UI; makes the segment publicly routable | yes |
| `layout.tsx` | shared shell, preserves state across navigation | yes |
| `template.tsx` | like a layout, but remounts on every navigation | yes |
| `loading.tsx` | Suspense fallback for the segment | no |
| `error.tsx` | error boundary for the segment — **must be a Client Component** | no |
| `global-error.tsx` | replaces the root layout when it throws | no |
| `not-found.tsx` | UI for `notFound()` and unmatched URLs | yes |
| `forbidden.tsx` | UI for `forbidden()` | yes |
| `unauthorized.tsx` | UI for `unauthorized()` | yes |
| `route.ts` | an HTTP endpoint; cannot coexist with `page.tsx` | yes |
| `default.tsx` | fallback for an unmatched parallel slot — **required in 16** | yes |
| `proxy.ts` | runs before a request is matched (was `middleware.ts`) | yes |
| `instrumentation.ts` | startup and observability hooks | yes |

| Folder syntax | Meaning |
|---|---|
| `[slug]` | dynamic segment → `{ slug: string }` |
| `[...slug]` | catch-all → `{ slug: string[] }` |
| `[[...slug]]` | optional catch-all → `{ slug?: string[] }` |
| `(group)` | route group — organises files, adds nothing to the URL |
| `@slot` | parallel route slot, passed to the layout as a prop |
| `(.)folder` | intercepting route — same level |
| `(..)folder` | one level up |
| `(...)folder` | from the root |
| `_folder` | private — opted out of routing entirely |

```
app/
  layout.tsx                 → wraps everything
  page.tsx                   → /
  loading.tsx                → fallback while / streams
  blog/
    layout.tsx               → wraps /blog and below
    page.tsx                 → /blog
    [slug]/
      page.tsx               → /blog/:slug
  (marketing)/
    about/page.tsx           → /about        the group is not in the URL
  api/
    items/route.ts           → /api/items
  _lib/
    db.ts                    → not routable
```

---

## Pages

A page is a default-exported component. Next generates a `PageProps<Route>` global for every route.

```tsx
// app/page.tsx
export default function Home() {
  return <h1>Home</h1>;
}
```

```tsx
// app/blog/[slug]/page.tsx
export default async function Post({ params, searchParams }: PageProps<"/blog/[slug]">) {
  const { slug } = await params;
  //     ^? string
  const { q } = await searchParams;
  //     ^? string | string[] | undefined

  const post = await getPost(slug);
  return <article><h1>{post.title}</h1></article>;
}

declare function getPost(slug: string): Promise<{ title: string }>;
```

The generated shape, verbatim from `.next/types/routes.d.ts`:

```ts
// quoted verbatim from .next/types/routes.d.ts — not standalone-compilable
interface PageProps<AppRoute extends AppRoutes> {
  params: Promise<ParamMap[AppRoute]>;
  searchParams: Promise<Record<string, string | string[] | undefined>>;
}

// and ParamMap is built from your actual folder structure:
interface ParamMap {
  "/": {};
  "/blog/[slug]": { slug: string };
}
```

`PageProps` is global — no import. The route string is checked against the routes that exist.

```tsx
// ✗ export default function P(props: PageProps<"/blogg/[slug]">) {}
//   Type '"/blogg/[slug]"' does not satisfy the constraint 'AppRoutes'.
```

### `params` and `searchParams` are promises

This is the headline breaking change of 15 → 16.

```tsx
// Next 16 — always await
export default async function P({ params }: PageProps<"/blog/[slug]">) {
  const { slug } = await params;
  return <p>{slug}</p>;
}

// ≤15 params was a plain object, and awaiting it was optional:
//   export default function P({ params }: { params: { slug: string } }) {
//     return <p>{params.slug}</p>;
//   }
//   Next 15 deprecated the synchronous form with a warning.
//   Next 16 REMOVED it: params is a promise, full stop.
//
// ✗ params.slug
//   Property 'slug' does not exist on type 'Promise<{ slug: string; }>'.

// in a Client Component, unwrap with React's use()
// "use client";
// import { use } from "react";
// export default function P({ params }: PageProps<"/blog/[slug]">) {
//   const { slug } = use(params);
//   return <p>{slug}</p>;
// }
```

```bash
# the codemod handles the mechanical cases
npx @next/codemod@canary next-async-request-api .
```

### Page-level route config

```ts
// exported consts, read at build time. They must be statically analysable —
// a computed value is a build error.
export const dynamic = "auto";            // "auto" | "force-dynamic" | "error" | "force-static"
export const revalidate = 3600;           // false | 0 | number (seconds)
export const fetchCache = "auto";
export const runtime = "nodejs";          // "nodejs" | "edge"
export const preferredRegion = "auto";
export const dynamicParams = true;        // allow params outside generateStaticParams
export const maxDuration = 10;
```

---

## Layouts

A layout wraps its segment and everything below it, and **does not re-render on navigation** within that subtree.

```tsx
// app/layout.tsx — the root layout is required and must render <html> and <body>
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: { default: "Site", template: "%s · Site" },
  description: "…",
};

export default function RootLayout({ children }: LayoutProps<"/">) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

```tsx
// app/blog/layout.tsx — a nested layout, with access to its own params
export default async function BlogLayout({ children, params }: LayoutProps<"/blog">) {
  await params;
  return <div className="prose">{children}</div>;
}
```

The generated shape:

```ts
// quoted verbatim from .next/types/routes.d.ts — not standalone-compilable
type LayoutProps<LayoutRoute extends LayoutRoutes> = {
  params: Promise<ParamMap[LayoutRoute]>;
  children: React.ReactNode;
} & {
  [K in LayoutSlotMap[LayoutRoute]]: React.ReactNode;   // parallel route slots
};
```

| | `layout.tsx` | `template.tsx` |
|---|---|---|
| remounts on navigation | no | yes |
| state preserved | yes | no |
| effects re-run | no | yes |
| use for | shells, nav, providers | enter/exit animations, per-view logging |

```tsx
// a layout does NOT receive searchParams — it is not re-rendered when they
// change. Read them in the page, or with useSearchParams in a client component.
// ✗ function L({ searchParams }: LayoutProps<"/">) {}
//   Property 'searchParams' does not exist on type 'LayoutProps<"/">'.
```

---

## Dynamic segments

```tsx
// app/shop/[category]/[item]/page.tsx  → /shop/shoes/boot
export default async function Item({ params }: PageProps<"/shop/[category]/[item]">) {
  const { category, item } = await params;
  //     ^? string          ^? string
  return <p>{category}/{item}</p>;
}
```

```tsx
// app/docs/[...slug]/page.tsx  → /docs/a/b/c
export default async function Docs({ params }: PageProps<"/docs/[...slug]">) {
  const { slug } = await params;
  //     ^? string[]                    → ['a', 'b', 'c']
  return <p>{slug.join("/")}</p>;
}
```

```tsx
// app/shop/[[...filters]]/page.tsx  → /shop  AND  /shop/a/b
export default async function Shop({ params }: PageProps<"/shop/[[...filters]]">) {
  const { filters } = await params;
  //     ^? string[] | undefined        → undefined at /shop
  return <p>{filters?.join(",") ?? "all"}</p>;
}
```

Every param value is a string — segments come from a URL. Parse and validate before use.

```tsx
export default async function P({ params }: PageProps<"/post/[id]">) {
  const { id } = await params;
  const n = Number(id);
  if (!Number.isInteger(n)) notFound();   // "abc" → 404, not NaN downstream
  return <p>{n}</p>;
}
declare function notFound(): never;
```

---

## Route groups

```
app/
  (marketing)/
    layout.tsx        → wraps /about and /pricing only
    about/page.tsx    → /about
    pricing/page.tsx  → /pricing
  (app)/
    layout.tsx        → a completely different shell
    dashboard/page.tsx → /dashboard
```

Groups let one URL tree have several root-level layouts. The folder name never appears in the URL, and never appears in `PageProps` route strings either.

```tsx
// the route string is the URL, not the folder path
export default function About(_: PageProps<"/about">) { return null; }
// ✗ PageProps<"/(marketing)/about">
//   does not satisfy the constraint 'AppRoutes'.
```

---

## Parallel and intercepting routes

```
app/
  layout.tsx          → receives `children`, `@team` and `@analytics` as props
  page.tsx
  @team/
    page.tsx
    default.tsx       → REQUIRED in Next 16
  @analytics/
    page.tsx
    default.tsx
```

```tsx
// app/layout.tsx — slots arrive as named props, typed by LayoutSlotMap
export default function Layout({
  children,
  team,
  analytics,
}: {
  children: React.ReactNode;
  team: React.ReactNode;
  analytics: React.ReactNode;
}) {
  return <>{children}<aside>{team}{analytics}</aside></>;
}
```

```tsx
// @team/default.tsx — what renders when the slot has no match for the URL
export default function Default() {
  return null;
}

// ≤15 a missing default.tsx fell back to rendering nothing on soft navigation
// and 404'd on hard navigation. Next 16 makes it a BUILD ERROR:
//   "Parallel route slot @team is missing a default.tsx"
// Return null, or call notFound(), but the file must exist.
```

Intercepting routes render a different UI for the same URL depending on how you arrived — the photo-modal pattern.

```
app/
  feed/page.tsx
  photo/[id]/page.tsx          → full page on a hard load
  feed/
    @modal/
      (..)photo/[id]/page.tsx  → a modal when navigated to from the feed
      default.tsx
```

---

## Typed routes

Stable in Next 16 (`typedRoutes: true`). Every internal `href` is checked against the routes that actually exist.

```ts
// next.config.ts
import type { NextConfig } from "next";
const config: NextConfig = { typedRoutes: true };
export default config;
```

```tsx
import Link from "next/link";
import type { Route } from "next";

<Link href="/blog/hello">ok</Link>;
<Link href="/blog/hello?tab=1">ok</Link>;
<Link href="/blog/hello#frag">ok</Link>;
<Link href="https://example.com">external, unchecked</Link>;

// ✗ <Link href="/blogg/hello">typo</Link>
//   Type '"/blogg/hello"' is not assignable to type 'UrlObject | RouteImpl<"/blogg/hello">'.

// a variable href must be typed as Route, or it widens to string and fails
const to = "/blog/hello" as Route;
<Link href={to}>ok</Link>;

// build one from a param safely
function postHref(slug: string) {
  return `/blog/${slug}` as Route;
}
<Link href={postHref("x")}>ok</Link>;
```

```ts
// the generated union, from .next/types/routes.d.ts — not standalone-compilable
type AppRoutes = "/" | "/blog/[slug]";
type Routes = AppRoutes | PageRoutes | LayoutRoutes | RedirectRoutes | RewriteRoutes;
export type ParamsOf<Route extends Routes> = ParamMap[Route];
```

To share a param type between a page and a helper, unwrap `PageProps` — it is
global, so this needs no import:

```ts
type PostParams = Awaited<PageProps<"/blog/[slug]">["params"]>;
//   ^? { slug: string }

function loadPost(p: PostParams) { return p.slug; }
```

`ParamsOf` is declared in the generated `.next/types/routes.d.ts`, but it is a
module export there rather than a global, so it is not importable from `"next"`:

```ts
// ✗ import type { ParamsOf } from "next";
//   Module '"next"' has no exported member 'ParamsOf'.
```

---

## Link

```tsx
import Link from "next/link";

<Link href="/about">About</Link>;

// prefetching
<Link href="/about" prefetch={false}>no prefetch</Link>;
<Link href="/about" prefetch>eager</Link>;
//  default: prefetches the route when the link enters the viewport

// replace instead of push
<Link href="/about" replace>replace</Link>;

// keep scroll position
<Link href="/about" scroll={false}>no scroll reset</Link>;

// an object href
<Link href={{ pathname: "/blog/[slug]", query: { slug: "a" } }}>obj</Link>;

// Link renders an <a>, so anchor props pass straight through
<Link href="/about" className="x" target="_blank" rel="noreferrer" aria-label="About" />;

// ≤12 you nested an <a> and needed legacyBehavior + passHref.
// Since 13 <Link> IS the anchor. Do not nest one.
// ✗ <Link href="/a"><a>text</a></Link>   → hydration error: <a> inside <a>
```

Next 16 improves navigation with layout deduplication and incremental prefetching, so prefetch payloads are much smaller on deep route trees.

---

## Navigation hooks

All from `next/navigation`, all client-only.

| Hook | Returns |
|---|---|
| `useRouter()` | `{ push, replace, refresh, back, forward, prefetch }` |
| `usePathname()` | `string` — the current path, no query |
| `useSearchParams()` | `ReadonlyURLSearchParams` |
| `useParams()` | the dynamic params, synchronously |
| `useSelectedLayoutSegment()` | `string \| null` |
| `useSelectedLayoutSegments()` | `string[]` |

```tsx
"use client";
import { useRouter, usePathname, useSearchParams, useParams } from "next/navigation";
import type { Route } from "next";

export function Nav() {
  const router = useRouter();
  const pathname = usePathname();
  //    ^? string                      → '/blog/hello'
  const search = useSearchParams();
  //    ^? ReadonlyURLSearchParams
  const params = useParams<{ slug: string }>();
  //    ^? { slug: string }            NOT a promise — this is the client hook

  const tab = search.get("tab");
  //    ^? string | null               → null when absent

  return (
    <button
      onClick={() => {
        const next = new URLSearchParams(search);   // copy: the original is readonly
        next.set("tab", "2");
        // under typedRoutes a template literal is not a known route, so cast:
        router.push(`${pathname}?${next}` as Route);
      }}
    >
      {tab ?? "none"}
    </button>
  );
}
```

```tsx
// the router methods
router.push("/about");
router.push("/about", { scroll: false });
router.replace("/about");
router.refresh();                   // re-fetch the current route's server data
router.back();
router.forward();
router.prefetch("/about");
declare const router: ReturnType<typeof import("next/navigation").useRouter>;

// ≤12 / pages/: these came from next/router and had a different shape
//   import { useRouter } from "next/router";
//   const { query, pathname, asPath, isReady } = useRouter();
// In the App Router `query` is split into useParams + useSearchParams, and
// there is no isReady — the hooks are correct on first render.
// ✗ importing next/router in an App Router component throws at runtime.
```

### `useSearchParams` opts a route into dynamic rendering

```tsx
// a component calling useSearchParams must sit inside a Suspense boundary,
// or the whole route becomes client-side rendered at build time.
import { Suspense } from "react";

declare function Nav(): React.ReactNode;

export default function Page() {
  return (
    <Suspense fallback={null}>
      <Nav />
    </Suspense>
  );
}
// build error otherwise:
//   useSearchParams() should be wrapped in a suspense boundary at page "/".
```

---

## Server vs client

Every component in `app/` is a **Server Component** unless a `"use client"` directive appears at the top of its file or of a file that imports it.

| | Server Component | Client Component |
|---|---|---|
| directive | none (default) | `"use client"` |
| may be `async` | yes | no |
| hooks (`useState`, `useEffect`) | no | yes |
| event handlers | no | yes |
| browser APIs | no | yes |
| `async`/`await` data access | yes | no |
| secrets, DB, filesystem | yes | never |
| ships JS to the browser | no | yes |
| `params` / `searchParams` | `await` them | `use()` them |

```tsx
// app/page.tsx — a Server Component: no directive needed
import { db } from "@/_lib/db";
import { Counter } from "./counter";

export default async function Page() {
  const rows = await db.query("select 1");    // safe: never reaches the client
  return <><ul>{rows.length}</ul><Counter start={0} /></>;
}
```

```tsx
// app/counter.tsx
"use client";
import { useState } from "react";

export function Counter({ start }: { start: number }) {
  const [n, setN] = useState(start);
  return <button onClick={() => setN((c) => c + 1)}>{n}</button>;
}
```

`"use client"` marks a **boundary**, not a file. Everything imported from a client module is also client code.

```tsx
// the composition rule: a Server Component may not be IMPORTED by a client
// component, but it may be PASSED to one as children.

// ✗ inside a "use client" file:
//   import ServerThing from "./server-thing";   → it becomes a client component

// ✓ pass it through from a server parent instead
// app/page.tsx (server)
export default function P() {
  return (
    <ClientShell>
      <ServerThing />         {/* rendered on the server, slotted in as children */}
    </ClientShell>
  );
}
declare function ClientShell(p: { children: React.ReactNode }): React.ReactNode;
declare function ServerThing(): React.ReactNode;
```

```tsx
// props crossing the boundary must be serialisable:
// primitives, plain objects, arrays, Date, Map, Set, FormData, promises,
// React elements, and Server Functions.
// ✗ <ClientThing onDone={() => {}} />  from a Server Component
//   Functions cannot be passed directly to Client Components.
// ✗ class instances, Symbols, functions

// a promise IS serialisable — pass it down and unwrap with use()
export default function Parent() {
  const data = getData();               // no await: start it, do not block
  return <ClientChild data={data} />;
}
declare function getData(): Promise<string>;
declare function ClientChild(p: { data: Promise<string> }): React.ReactNode;

// "server-only" and "client-only" make an accidental import a build error
// import "server-only";   at the top of a module with secrets
// import "client-only";   at the top of a module using window
```

---

## Request APIs

All async in Next 16. All opt the route into dynamic rendering.

```tsx
import { cookies, headers, draftMode } from "next/headers";
import { connection, after } from "next/server";

export default async function Page() {
  const cookieStore = await cookies();
  //    ^? ReadonlyRequestCookies
  const token = cookieStore.get("token")?.value;
  //    ^? string | undefined
  cookieStore.getAll();
  cookieStore.has("token");             // → boolean

  const h = await headers();
  //    ^? ReadonlyHeaders
  const ua = h.get("user-agent");
  //    ^? string | null

  const { isEnabled } = await draftMode();
  //      ^? boolean

  return <p>{token ?? "anon"}</p>;
}
```

```tsx
// ≤15 these were synchronous, then deprecated-but-working:
//   const token = cookies().get("token")?.value;
// Next 16 removed the sync form entirely.
// ✗ cookies().get("token")
//   Property 'get' does not exist on type 'Promise<ReadonlyRequestCookies>'.
```

Cookies are **writable only in a Server Function or a route handler** — never during a render.

```ts
"use server";
import { cookies } from "next/headers";

export async function login(token: string) {
  const store = await cookies();
  store.set("token", token, { httpOnly: true, secure: true, sameSite: "lax", maxAge: 3600 });
  store.delete("stale");
}
// ✗ calling store.set() from a page component:
//   Cookies can only be modified in a Server Action or Route Handler.
```

### `connection()` and `after()`

```tsx
import { connection, after } from "next/server";

export default async function Page() {
  await connection();
  //    marks everything below as requiring a real request. Use it when a route
  //    is dynamic for a reason Next cannot see — Math.random(), Date.now().
  const n = Math.random();

  after(() => {
    //  runs AFTER the response is sent: logging, analytics, cache warming.
    //  Does not delay the user.
    void log(n);
  });

  return <p>{n}</p>;
}
declare function log(n: number): Promise<void>;

// ≤15 `unstable_after` behind a flag, and `unstable_noStore()` instead of
// connection(). Both are stable and renamed in 16.
```

---

## Data fetching

Fetch where the data is used. There is no `getServerSideProps` in the App Router.

```tsx
// the normal case: await in the component that needs it
export default async function Page() {
  const res = await fetch("https://api.example.com/items");
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  const items = (await res.json()) as Item[];
  //    ^? Item[]                 json() returns any — always assert or validate
  return <ul>{items.map((i) => <li key={i.id}>{i.name}</li>)}</ul>;
}
type Item = { id: string; name: string };
```

```tsx
// pages/ equivalents, for orientation:
//   getServerSideProps  → just await in the component (dynamic)
//   getStaticProps      → await in the component + "use cache" (static)
//   getStaticPaths      → generateStaticParams
//   getInitialProps     → removed; no equivalent, and it was always discouraged
```

### `fetch` options

```tsx
// Next 16: fetch is NOT cached by default — the same as plain fetch.
await fetch(url);                                   // dynamic, no cache
await fetch(url, { cache: "force-cache" });         // cached indefinitely
await fetch(url, { cache: "no-store" });            // explicit opt out
await fetch(url, { next: { revalidate: 3600 } });   // ISR: refresh hourly
await fetch(url, { next: { tags: ["items"] } });    // taggable for revalidateTag
declare const url: string;

// ≤14 fetch WAS cached by default ("force-cache"), which surprised everyone.
// Next 15 flipped the default to no-store. If you are porting 13/14 code,
// every fetch you relied on caching now needs an explicit option.
```

### Parallel vs sequential

```tsx
// sequential — two round trips, the second waits for the first
async function slow(id: string) {
  const user = await getUser(id);
  const posts = await getPosts(id);
  return { user, posts };
}

// parallel — start both, then await
async function fast(id: string) {
  const userP = getUser(id);
  const postsP = getPosts(id);
  const [user, posts] = await Promise.all([userP, postsP]);
  return { user, posts };
}

declare function getUser(id: string): Promise<{ id: string }>;
declare function getPosts(id: string): Promise<string[]>;

// better still: do not await at all — pass the promise to a child inside
// its own Suspense boundary, so the page streams.
```

### Request deduplication

```tsx
import { cache } from "react";

// React's cache() dedupes calls within a single render pass, so several
// components can each ask for the same thing without N queries.
export const getUserCached = cache(async (id: string) => {
  return db.user(id);
});
declare const db: { user(id: string): Promise<{ id: string }> };

// fetch() to the same URL with the same options is deduped automatically.
// cache() is for everything else: database calls, filesystem reads.
```

---

## use cache

The Next 16 caching model. Mark a function, file or component with `"use cache"` and its output is cached.

```ts
// next.config.ts
import type { NextConfig } from "next";
const config: NextConfig = { cacheComponents: true };
export default config;

// ≤15 this was experimental.ppr / experimental.dynamicIO / experimental.useCache.
// Next 16 removed experimental.ppr; cacheComponents replaces it.
```

```tsx
import { cacheLife, cacheTag } from "next/cache";

// a cached function
async function getItems() {
  "use cache";
  cacheLife("hours");
  cacheTag("items");
  const res = await fetch("https://api.example.com/items");
  return (await res.json()) as Item[];
}
type Item = { id: string; name: string };

// a cached component — its rendered output is cached, not just its data
async function ItemList() {
  "use cache";
  cacheLife("minutes");
  const items = await getItems();
  return <ul>{items.map((i) => <li key={i.id}>{i.name}</li>)}</ul>;
}

// a whole cached file: put "use cache" at the top, above the imports
```

### `cacheLife` profiles

The built-in profiles, from the shipped type definitions:

| Profile | stale | revalidate | expire |
|---|---|---|---|
| `"default"` | 5 min | 15 min | never |
| `"seconds"` | 30 s | 1 s | 1 min |
| `"minutes"` | 5 min | 1 min | 1 hour |
| `"hours"` | 5 min | 1 hour | 1 day |
| `"days"` | 5 min | 1 day | 1 week |
| `"weeks"` | 5 min | 1 week | 30 days |
| `"max"` | 5 min | 30 days | never |

```ts
// stale      — how long a client may serve it without asking the server
// revalidate — after this, the server refreshes it in the background
// expire     — with no traffic for this long, it is dropped and recomputed

cacheLife("hours");
cacheLife({ stale: 60, revalidate: 300, expire: 3600 });   // a custom timespan
declare function cacheLife(p: string | { stale?: number; revalidate?: number; expire?: number }): void;

// named custom profiles go in next.config.ts under cacheLife: { ... }
```

### Arguments must be serialisable

```tsx
// a "use cache" function's arguments form its cache key, so they must
// serialise — and its return value must too.
async function getPost(slug: string) {
  "use cache";
  return { slug };
}
getPost("a");                       // fine

// ✗ passing a function, a class instance or a Symbol → build error
// ✗ reading cookies() or headers() inside "use cache" → they are request-scoped
//   and a cache entry is shared across requests.
//   Read them OUTSIDE and pass the value in as an argument.
```

---

## Revalidation

Four functions, from `next/cache`, with genuinely different semantics.

| Function | Effect | Callable from |
|---|---|---|
| `revalidateTag(tag, profile)` | marks a tag stale; next request refreshes | Server Function, route handler |
| `updateTag(tag)` | expires immediately, read-your-own-writes | **Server Functions only** |
| `revalidatePath(path, type?)` | invalidates a path | Server Function, route handler |
| `refresh()` | refreshes client-side cached data | Server Function |

```ts
"use server";
import { revalidateTag, updateTag, revalidatePath, refresh } from "next/cache";

export async function publish(id: string) {
  await db.publish(id);

  // Next 16: the profile argument is REQUIRED
  revalidateTag("posts", "max");
  revalidateTag("posts", { expire: 3600 });

  // ≤15: revalidateTag("posts")   — one argument
  // ✗ revalidateTag("posts")
  //   Expected 2 arguments, but got 1.

  // when the user must see their own write on the very next render,
  // revalidateTag is not enough — it only marks stale
  updateTag("posts");

  revalidatePath("/blog");
  revalidatePath("/blog/[slug]", "page");
  revalidatePath("/", "layout");      // the whole subtree

  refresh();                          // drop client-side cached data too
}
declare const db: { publish(id: string): Promise<void> };
```

`revalidateTag` vs `updateTag` is the distinction to internalise: `revalidateTag` says "this is stale, refresh it whenever"; `updateTag` says "I just wrote this, show it to me now". Using the first after a mutation is the cause of the classic "I saved but the list still shows the old value" bug.

```ts
// time-based revalidation, for a whole route
export const revalidate = 3600;     // seconds; false = never, 0 = always dynamic
```

---

## Server Functions

A function marked `"use server"` runs on the server and is callable from the client as if it were local. Next generates the endpoint.

```ts
// app/_lib/actions.ts
"use server";
import { z } from "zod";
import { revalidateTag, updateTag } from "next/cache";
import { redirect } from "next/navigation";

const CreateSchema = z.object({
  title: z.string().min(1, "title required"),
  body: z.string().min(1),
});

export type ActionState =
  | { status: "idle" }
  | { status: "error"; message: string; fieldErrors?: Record<string, string[]> }
  | { status: "success"; id: string };

export async function createPost(
  _prev: ActionState,
  formData: FormData,
): Promise<ActionState> {
  const parsed = CreateSchema.safeParse({
    title: formData.get("title"),
    body: formData.get("body"),
  });

  if (!parsed.success) {
    return {
      status: "error",
      message: "invalid input",
      fieldErrors: parsed.error.flatten().fieldErrors,
    };
  }

  const id = await db.create(parsed.data);
  updateTag("posts");
  redirect(`/blog/${id}`);          // throws; never returns
}

declare const db: { create(v: { title: string; body: string }): Promise<string> };
```

```ts
// two placements of the directive:
//   at the top of a FILE  → every export is a Server Function
//   at the top of a FUNCTION BODY → only that function
export default async function Page() {
  async function inline(formData: FormData) {
    "use server";
    console.log(formData.get("x"));
  }
  return <form action={inline}><input name="x" /></form>;
}
```

| Rule | Detail |
|---|---|
| must be `async` | a sync function marked `"use server"` is a build error |
| arguments and return must serialise | same rules as client props |
| it is a **public HTTP endpoint** | authenticate and authorise inside it, every time |
| never trust the arguments | validate with a schema; the client controls them |
| closed-over variables are serialised | and sent to the client encrypted; keep them small |

```ts
// ✗ export function sync() { "use server"; }
//   Server Actions must be async functions.

// the security point, spelled out: an exported Server Function is reachable by
// anyone who can POST to your site. This is not "internal".
"use server";
export async function deletePost(id: string) {
  const user = await requireUser();          // ← not optional
  if (!user.canDelete) throw new Error("forbidden");
  await db2.delete(id);
}
declare function requireUser(): Promise<{ canDelete: boolean }>;
declare const db2: { delete(id: string): Promise<void> };
```

### Calling one from a client component

```tsx
"use client";
import { useTransition } from "react";
import { deletePost } from "@/_lib/actions";

export function DeleteButton({ id }: { id: string }) {
  const [pending, start] = useTransition();
  return (
    <button
      disabled={pending}
      onClick={() => start(async () => { await deletePost(id); })}
    >
      {pending ? "…" : "Delete"}
    </button>
  );
}
// the import looks local; the call is an HTTP POST. Nothing of the function
// body is shipped to the browser.
```

### Binding extra arguments

```tsx
import { deletePost } from "@/_lib/actions";

// .bind is the safe way to pass an id into a form action — the value is not
// rendered into the DOM as a hidden input the user can edit
export function Row({ id }: { id: string }) {
  const boundDelete = deletePost.bind(null, id);
  return <form action={boundDelete}><button>Delete</button></form>;
}
```

---

## Forms

The full pattern, combining Next's Server Functions with React 19's form hooks.

```tsx
// app/new/form.tsx
"use client";
import { useActionState } from "react";
import { useFormStatus } from "react-dom";
import { createPost, type ActionState } from "@/_lib/actions";

const initial: ActionState = { status: "idle" };

export function NewPostForm() {
  const [state, action, isPending] = useActionState(createPost, initial);
  //     ^? ActionState   ^? (fd: FormData) => void   ^? boolean

  return (
    <form action={action}>
      <input name="title" />
      {state.status === "error" && state.fieldErrors?.title && (
        <p role="alert">{state.fieldErrors.title.join(", ")}</p>
      )}
      <textarea name="body" />
      <Submit />
      {state.status === "error" && <p role="alert">{state.message}</p>}
    </form>
  );
}

function Submit() {
  const { pending } = useFormStatus();     // must be INSIDE the <form>
  return <button disabled={pending}>{pending ? "Saving…" : "Save"}</button>;
}
```

The `ActionState` discriminated union is doing real work: `fieldErrors` is only reachable in the error branch, so the template cannot accidentally read it on success.

```tsx
// a plain server-rendered form, no client JS at all — works before hydration.
// NOTE: a useActionState action returns state, so it is NOT a valid bare
// `action`, which must resolve to void. Write a second, void-returning entry
// point rather than reusing createPost:
import { createPostAndRedirect } from "@/_lib/actions";

export default function Page() {
  return (
    <form action={createPostAndRedirect}>
      <input name="title" />
      <button>Save</button>
    </form>
  );
}

// ✗ <form action={createPost.bind(null, { status: "idle" })}>
//   Type '(f: FormData) => Promise<ActionState>' is not assignable to type
//   'string | ((formData: FormData) => void | Promise<void>) | undefined'.
```

### `next/form`

```tsx
import Form from "next/form";

// a <form> that navigates instead of posting: prefetches the target route,
// does a client-side navigation, and preserves shared layout state
<Form action="/search">
  <input name="q" />
  <button>Search</button>
</Form>;
// submitting sends the user to /search?q=… without a full page load
```

Use `next/form` for GET-style search forms, and `action={serverFunction}` for mutations.

---

## Route handlers

`app/**/route.ts` exports one function per HTTP method. Uses the Web `Request`/`Response` APIs.

```ts
// app/api/items/route.ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export async function GET(request: NextRequest) {
  const q = request.nextUrl.searchParams.get("q");
  //    ^? string | null
  const items = await db.list(q);
  return NextResponse.json(items);
  //     ^? NextResponse<Item[]>
}

export async function POST(request: NextRequest) {
  const body = (await request.json()) as { name?: unknown };
  if (typeof body.name !== "string") {
    return NextResponse.json({ error: "name required" }, { status: 400 });
  }
  const created = await db.create(body.name);
  return NextResponse.json(created, { status: 201 });
}

type Item = { id: string; name: string };
declare const db: {
  list(q: string | null): Promise<Item[]>;
  create(n: string): Promise<Item>;
};
```

Supported exports: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `HEAD`, `OPTIONS`.

```ts
// app/api/items/[id]/route.ts — the generated RouteContext gives typed params
export async function GET(request: Request, ctx: RouteContext<"/api/items/[id]">) {
  const { id } = await ctx.params;
  //     ^? string                  — a promise here too, as of 15
  return Response.json({ id });
}

// the generated shape:
//   interface RouteContext<R extends AppRouteHandlerRoutes> {
//     params: Promise<ParamMap[R]>
//   }
```

```ts
// ≤15 / pages/: the old API routes had Node req/res objects
//   // pages/api/items.ts
//   import type { NextApiRequest, NextApiResponse } from "next";
//   export default function handler(req: NextApiRequest, res: NextApiResponse) {
//     res.status(200).json({ ok: true });
//   }
// Route handlers are Web-standard instead: return a Response, do not mutate one.
```

### Responses beyond JSON

```ts
import { NextResponse } from "next/server";

// plain text / html
new Response("hello", { headers: { "content-type": "text/plain" } });

// redirect
NextResponse.redirect(new URL("/login", "https://example.com"));

// cookies on the response
const res = NextResponse.json({ ok: true });
res.cookies.set("token", "abc", { httpOnly: true });
res.headers.set("cache-control", "no-store");

// streaming
const stream = new ReadableStream({
  start(controller) {
    controller.enqueue(new TextEncoder().encode("chunk"));
    controller.close();
  },
});
new Response(stream, { headers: { "content-type": "text/event-stream" } });

// route segment config works here too
export const dynamic = "force-dynamic";
export const runtime = "nodejs";
```

```ts
// route.ts and page.tsx cannot live in the same folder
// ✗ app/items/page.tsx + app/items/route.ts
//   "Conflicting route and page at /items"
```

---

## proxy.ts

Runs before a request is matched to a route. Renamed from `middleware.ts` in Next 16.

```ts
// proxy.ts — at the project root, beside app/
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function proxy(request: NextRequest) {
  const token = request.cookies.get("token")?.value;

  if (!token && request.nextUrl.pathname.startsWith("/dashboard")) {
    const url = request.nextUrl.clone();
    url.pathname = "/login";
    url.searchParams.set("from", request.nextUrl.pathname);
    return NextResponse.redirect(url);
  }

  const res = NextResponse.next();
  res.headers.set("x-request-id", crypto.randomUUID());
  return res;
}

export const config = {
  matcher: [
    "/dashboard/:path*",
    // everything except static assets and images
    "/((?!_next/static|_next/image|favicon.ico).*)",
  ],
};
```

```ts
// ≤15 the file was middleware.ts and the export was `middleware`:
//   // middleware.ts
//   export function middleware(request: NextRequest) { ... }
//
// The logic is identical — only the file name and the exported function name
// changed. middleware.ts still works in 16 but is deprecated. The new name
// reflects what it is: the network boundary, not a general request hook.
//
// codemod:
//   npx @next/codemod@canary middleware-to-proxy .
```

| Rule | Detail |
|---|---|
| runs on every matched request | keep it fast; it is in the critical path |
| Edge runtime by default | no Node built-ins unless configured |
| cannot query a database | do auth checks on a token, not a session lookup |
| `matcher` must be statically analysable | a computed array is ignored |
| one `proxy.ts` per project | at the root, or inside `src/` |

Use it for redirects, rewrites, headers and cheap token checks. Real authorisation belongs in the page, the Server Function and the route handler — anything else is a bypassable check.

---

## Metadata

```tsx
// static — a plain exported object
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: "About",
  description: "About us",
  openGraph: {
    title: "About",
    images: [{ url: "/og.png", width: 1200, height: 630 }],
  },
  twitter: { card: "summary_large_image" },
  robots: { index: true, follow: true },
  alternates: { canonical: "https://example.com/about" },
};
```

```tsx
// dynamic — an async function, deduped against the page's own fetches
import type { Metadata } from "next";

export async function generateMetadata(
  { params }: PageProps<"/blog/[slug]">,
  parent: import("next").ResolvingMetadata,
): Promise<Metadata> {
  const { slug } = await params;
  const post = await getPost(slug);
  const parentOg = (await parent).openGraph;

  return {
    title: post.title,
    description: post.excerpt,
    openGraph: { ...parentOg, title: post.title },
  };
}

export default async function Page({ params }: PageProps<"/blog/[slug]">) {
  const { slug } = await params;
  const post = await getPost(slug);         // deduped with the call above
  return <h1>{post.title}</h1>;
}

declare function getPost(s: string): Promise<{ title: string; excerpt: string }>;
```

```tsx
import type { Metadata } from "next";

// title templates, set on a layout and applied to descendants
export const metadata: Metadata = {
  title: {
    default: "Site",          // used when a child sets no title
    template: "%s · Site",    // "About" → "About · Site"
    absolute: undefined,      // a child can set `absolute` to ignore the template
  },
};
```

| File convention | Generates |
|---|---|
| `app/favicon.ico` | the favicon |
| `app/icon.(png\|svg)` | `<link rel="icon">` |
| `app/apple-icon.png` | apple touch icon |
| `app/opengraph-image.(png\|tsx)` | `og:image` |
| `app/twitter-image.(png\|tsx)` | twitter card image |
| `app/robots.ts` | `/robots.txt` |
| `app/sitemap.ts` | `/sitemap.xml` |
| `app/manifest.ts` | `/manifest.webmanifest` |

```ts
// app/sitemap.ts
import type { MetadataRoute } from "next";

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const posts = await listPosts();
  return [
    { url: "https://example.com", lastModified: new Date(), priority: 1 },
    ...posts.map((p) => ({
      url: `https://example.com/blog/${p.slug}`,
      lastModified: p.updatedAt,
    })),
  ];
}
declare function listPosts(): Promise<{ slug: string; updatedAt: Date }[]>;
```

```ts
// app/robots.ts
import type { MetadataRoute } from "next";
export default function robots(): MetadataRoute.Robots {
  return {
    rules: { userAgent: "*", allow: "/", disallow: "/admin" },
    sitemap: "https://example.com/sitemap.xml",
  };
}
```

```tsx
// app/opengraph-image.tsx — a generated image, typed as a React tree
import { ImageResponse } from "next/og";

export const size = { width: 1200, height: 630 };
export const contentType = "image/png";

export default async function Image() {
  return new ImageResponse(
    <div style={{ display: "flex", fontSize: 64, background: "#fff", width: "100%", height: "100%" }}>
      Hello
    </div>,
    size,
  );
}
// only a flexbox subset of CSS is supported. `display: flex` is required on
// any element with more than one child — this is the most common failure.
// ImageResponse is 2–20x faster as of 16.2.
```

Metadata exports are ignored in Client Components. Use React 19's hoisted `<title>` / `<meta>` tags there instead, or lift the metadata to a server parent.

---

## generateStaticParams

Pre-renders dynamic routes at build time. Replaces `getStaticPaths`.

```tsx
// app/blog/[slug]/page.tsx
export async function generateStaticParams() {
  const posts = await listPosts();
  return posts.map((p) => ({ slug: p.slug }));
  //     ^? { slug: string }[]      — values must be strings, always
}

export default async function Page({ params }: PageProps<"/blog/[slug]">) {
  const { slug } = await params;
  return <h1>{slug}</h1>;
}

declare function listPosts(): Promise<{ slug: string }[]>;
```

```tsx
// catch-all routes take arrays
// app/docs/[...slug]/page.tsx
export async function generateStaticParams() {
  return [{ slug: ["a", "b"] }, { slug: ["c"] }];
  // → prerenders /docs/a/b and /docs/c
}

// nested dynamic segments: return every combination (in its own file —
// one generateStaticParams per route)
// app/[lang]/blog/[slug]/page.tsx
//   export async function generateStaticParams() {
//     return [
//       { lang: "en", slug: "hello" },
//       { lang: "fr", slug: "bonjour" },
//     ];
//   }

// ✗ return posts.map((p) => ({ id: p.id }));   where id is a number
//   params must be strings. Use String(p.id).
```

```ts
// what happens for a param NOT in the list
export const dynamicParams = true;    // default: render on demand, then cache
//           dynamicParams = false    // 404 instead

// pages/ equivalent:
//   export async function getStaticPaths() {
//     return { paths: [{ params: { slug: "a" } }], fallback: "blocking" };
//   }
//   fallback: false      → dynamicParams = false
//   fallback: "blocking" → dynamicParams = true
//   fallback: true       → no direct equivalent; use loading.tsx
```

---

## Streaming

```tsx
// app/loading.tsx — an automatic Suspense boundary for the whole segment
export default function Loading() {
  return <p>Loading…</p>;
}
// Next wraps page.tsx in <Suspense fallback={<Loading />}> for you.
```

```tsx
// finer control: stream the slow part only, and show the rest immediately
import { Suspense } from "react";

export default function Page() {
  return (
    <>
      <Header />                                  {/* instant */}
      <Suspense fallback={<FeedSkeleton />}>
        <Feed />                                  {/* streams in */}
      </Suspense>
      <Suspense fallback={<SidebarSkeleton />}>
        <Sidebar />                               {/* independently */}
      </Suspense>
    </>
  );
}

async function Feed() {
  const items = await slowQuery();
  return <ul>{items.map((i) => <li key={i}>{i}</li>)}</ul>;
}

declare function Header(): React.ReactNode;
declare function FeedSkeleton(): React.ReactNode;
declare function SidebarSkeleton(): React.ReactNode;
declare function Sidebar(): React.ReactNode;
declare function slowQuery(): Promise<string[]>;
```

The rule: **the await location decides what blocks**. An `await` in the page body blocks the whole page; an `await` inside a component under `<Suspense>` blocks only that component.

```tsx
import { Suspense } from "react";

declare function slowQuery(): Promise<string[]>;

// ✗ this defeats streaming — the await is above the boundary
async function Bad() {
  const items = await slowQuery();
  return <Suspense fallback={<p>…</p>}><ul>{items.length}</ul></Suspense>;
}

// ✓ push the await down, or pass the promise
function Good() {
  const itemsP = slowQuery();                     // no await
  return (
    <Suspense fallback={<p>…</p>}>
      <Items itemsP={itemsP} />
    </Suspense>
  );
}

async function Items({ itemsP }: { itemsP: Promise<string[]> }) {
  const items = await itemsP;
  return <ul>{items.length}</ul>;
}
```

With `cacheComponents: true`, Next serves the cached shell instantly and streams the dynamic holes — partial pre-rendering, without the `experimental.ppr` flag that 16 removed.

---

## Errors

```tsx
// app/error.tsx — MUST be a client component
"use client";
import { useEffect } from "react";

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => { report(error); }, [error]);

  return (
    <div role="alert">
      <p>{error.message}</p>
      <button onClick={() => reset()}>Try again</button>
    </div>
  );
}
declare function report(e: Error): void;
```

```tsx
// app/global-error.tsx — catches errors in the ROOT layout, so it must
// render <html> and <body> itself
"use client";
export default function GlobalError({ error, reset }: { error: Error; reset: () => void }) {
  return (
    <html><body>
      <h1>Something broke</h1>
      <button onClick={reset}>Retry</button>
    </body></html>
  );
}
```

| Boundary | Catches |
|---|---|
| `error.tsx` | render errors in its segment and below — **not** its own layout |
| `global-error.tsx` | errors in the root layout; only active in production |
| `not-found.tsx` | `notFound()` and unmatched URLs |
| `forbidden.tsx` | `forbidden()` |
| `unauthorized.tsx` | `unauthorized()` |

```tsx
// error.tsx does NOT catch errors thrown by the layout at the same level.
// app/blog/error.tsx catches app/blog/page.tsx, but not app/blog/layout.tsx —
// for that you need app/error.tsx one level up.
```

```ts
// in production, the error message is redacted and replaced by a digest,
// so never rely on error.message for user-facing text. Model expected
// failures as return values, and let errors mean "a bug happened".
```

---

## redirect and notFound

All from `next/navigation`. All **throw** — they never return, and their type is `never`.

```tsx
import { redirect, permanentRedirect, notFound, forbidden, unauthorized } from "next/navigation";

export default async function Page({ params }: PageProps<"/blog/[slug]">) {
  const { slug } = await params;
  const user = await getUser();

  if (!user) redirect("/login");        // 307 (or 303 in a Server Function)
  if (!user.verified) unauthorized();   // 401 → app/unauthorized.tsx
  if (!user.canRead) forbidden();       // 403 → app/forbidden.tsx

  const post = await findPost(slug);
  if (!post) notFound();                // 404 → app/not-found.tsx

  // TypeScript knows post is non-null here: notFound() returns never
  return <h1>{post.title}</h1>;
  //        ^? post is { title: string }
}

declare function getUser(): Promise<{ verified: boolean; canRead: boolean } | null>;
declare function findPost(s: string): Promise<{ title: string } | null>;
```

```ts
import { permanentRedirect } from "next/navigation";

// permanentRedirect is 308 — cached by browsers essentially forever.
// Use redirect() unless you are certain.
permanentRedirect("/new-home");

// the never return type is what makes the narrowing above work:
//   declare function notFound(): never;
```

```ts
import { redirect } from "next/navigation";

// ✗ calling redirect() inside try/catch swallows it
try {
  redirect("/login");                   // throws a special NEXT_REDIRECT error
} catch (e) {
  // ...which this catch block now eats, and the redirect never happens
}
// → call redirect() AFTER the try/catch, or rethrow.

// forbidden() and unauthorized() require the authInterrupts flag:
//   experimental: { authInterrupts: true }
```

---

## Images

```tsx
import Image from "next/image";
import hero from "@/public/hero.png";     // a static import: dimensions inferred

// static import — width, height and blur placeholder come for free
<Image src={hero} alt="Hero" placeholder="blur" priority />;

// remote — width and height are REQUIRED, or use fill
<Image src="https://cdn.example.com/a.jpg" alt="A" width={800} height={600} />;

// fill: the parent must be positioned and sized
<div style={{ position: "relative", width: 300, height: 200 }}>
  <Image src="/b.jpg" alt="B" fill style={{ objectFit: "cover" }} sizes="300px" />
</div>;

// sizes matters for responsive images — without it the browser downloads
// the largest candidate
<Image src="/c.jpg" alt="C" fill sizes="(max-width: 768px) 100vw, 50vw" />;
```

```ts
// next.config.ts — remote hosts must be allowlisted
import type { NextConfig } from "next";
const config: NextConfig = {
  images: {
    remotePatterns: [
      { protocol: "https", hostname: "cdn.example.com", pathname: "/**" },
    ],
    qualities: [75],            // 16: default is [75]; other values now error
    minimumCacheTTL: 14400,     // 16: default raised from 60s to 4 hours
  },
};
export default config;

// ✗ <Image src="https://other.com/x.jpg" ... />
//   Invalid src prop, hostname "other.com" is not configured under images

// ✗ <Image src="/a.jpg" quality={90} />   in Next 16
//   quality 90 is not configured in images.qualities
```

`priority` on the largest above-the-fold image, `alt` always (it is required by the types), `sizes` whenever you use `fill`.

---

## Fonts

```tsx
// app/fonts.ts
import { Inter, Roboto_Mono } from "next/font/google";
import localFont from "next/font/local";

export const inter = Inter({
  subsets: ["latin"],
  display: "swap",
  variable: "--font-inter",
});

export const mono = Roboto_Mono({ subsets: ["latin"], variable: "--font-mono" });

export const custom = localFont({
  src: [
    { path: "./fonts/a.woff2", weight: "400", style: "normal" },
    { path: "./fonts/a-bold.woff2", weight: "700", style: "normal" },
  ],
  variable: "--font-custom",
});
```

```tsx
// app/layout.tsx
import { inter, mono } from "./fonts";

export default function Layout({ children }: LayoutProps<"/">) {
  return (
    <html lang="en" className={`${inter.variable} ${mono.variable}`}>
      <body className={inter.className}>{children}</body>
    </html>
  );
}
```

Fonts must be called at module scope with literal arguments — they are resolved at build time and self-hosted, so there is no request to Google at runtime.

```ts
// ✗ const f = Inter({ subsets: [sub] });   where sub is a variable
//   Font loader values must be explicitly written literals.
```

---

## Scripts

```tsx
import Script from "next/script";

<Script src="https://example.com/a.js" strategy="afterInteractive" />;
<Script src="https://example.com/b.js" strategy="lazyOnload" />;
<Script src="https://example.com/c.js" strategy="beforeInteractive" />;
<Script id="inline" strategy="afterInteractive">{`console.log("hi")`}</Script>;
```

| `strategy` | Loads |
|---|---|
| `beforeInteractive` | before hydration — root layout only |
| `afterInteractive` | after hydration (default) |
| `lazyOnload` | during browser idle |
| `worker` | in a web worker (experimental) |

---

## Environment variables

```ts
// server-only — never sent to the browser
process.env.DATABASE_URL;
//          ^? string | undefined        always, so validate at startup

// exposed to the browser: the NEXT_PUBLIC_ prefix, inlined at BUILD time
process.env.NEXT_PUBLIC_API_URL;

// ✗ process.env[key]  — dynamic access is not inlined and returns undefined
//   in client code. The replacement is textual and needs a literal.
```

```ts
// env.ts — validate once, import the typed object everywhere
import { z } from "zod";

const Env = z.object({
  DATABASE_URL: z.string().url(),
  NEXT_PUBLIC_API_URL: z.string().url(),
  NODE_ENV: z.enum(["development", "production", "test"]),
});

export const env = Env.parse(process.env);
//           ^? { DATABASE_URL: string; NEXT_PUBLIC_API_URL: string; NODE_ENV: ... }
```

| File | Loaded in |
|---|---|
| `.env` | every environment |
| `.env.local` | every environment except test; **gitignored** |
| `.env.development` / `.env.production` | that mode |
| `.env.development.local` | that mode, local overrides |

```ts
// ≤15 you could also use serverRuntimeConfig / publicRuntimeConfig.
// Next 16 REMOVED both. Use .env files.
// ✗ import getConfig from "next/config";
//   Module has no exported member.
```

---

## next.config.ts

```ts
import type { NextConfig } from "next";

const config: NextConfig = {
  typedRoutes: true,
  cacheComponents: true,              // 16: replaces experimental.ppr

  reactCompiler: true,                // 16: stable React Compiler support

  images: {
    remotePatterns: [{ protocol: "https", hostname: "cdn.example.com" }],
  },

  async redirects() {
    return [{ source: "/old", destination: "/new", permanent: true }];
  },

  async rewrites() {
    return [{ source: "/proxy/:path*", destination: "https://api.example.com/:path*" }];
  },

  async headers() {
    return [{
      source: "/(.*)",
      headers: [{ key: "x-frame-options", value: "DENY" }],
    }];
  },

  typescript: { ignoreBuildErrors: false },   // never set this to true
  // ≤15 also had `eslint: { ignoreDuringBuilds }`. Next 16 removed `next lint`
  // and the key with it:
  // ✗ eslint: { ignoreDuringBuilds: false }
  //   Object literal may only specify known properties, and 'eslint' does not
  //   exist in type 'NextConfig'.
};

export default config;
```

`next.config.ts` has been supported since 15. `NextConfig` catches typos in keys, which is most of the value.

---

## General idioms

### A typed data layer

```ts
// app/_lib/posts.ts
import "server-only";                  // a client import of this file now fails
import { cache } from "react";
import { z } from "zod";

const Post = z.object({
  id: z.string(),
  title: z.string(),
  publishedAt: z.coerce.date(),
});

export type Post = z.infer<typeof Post>;
//          ^? { id: string; title: string; publishedAt: Date }

export const getPost = cache(async (slug: string): Promise<Post | null> => {
  const res = await fetch(`https://api.example.com/posts/${slug}`, {
    next: { tags: [`post:${slug}`], revalidate: 3600 },
  });
  if (res.status === 404) return null;
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return Post.parse(await res.json());
});
```

One schema, one inferred type, one tagged fetch, deduped per render. Every page that needs a post calls `getPost` and gets the same guarantees.

### Route-aware helpers

```ts
import type { Route } from "next";

// centralise URL construction so a route rename is one edit
export const routes = {
  home: () => "/" as Route,
  post: (slug: string) => `/blog/${slug}` as Route,
  editPost: (id: string) => `/admin/posts/${id}` as Route,
};
```

### Loading and empty states as data

```tsx
// model the three outcomes, rather than rendering from two booleans
export default async function Page({ params }: PageProps<"/blog/[slug]">) {
  const { slug } = await params;
  const post = await getPost(slug);
  if (!post) notFound();
  return <article>{post.title}</article>;
}
declare function getPost(s: string): Promise<{ title: string } | null>;
declare function notFound(): never;
```

### Keeping secrets server-side

```ts
// the three defences, in order of reliability:
//   1. import "server-only" in the module         → build error on client import
//   2. no NEXT_PUBLIC_ prefix on the env var      → not inlined into the bundle
//   3. read it inside a Server Function or route handler, never a shared module
//
// what does NOT work: assuming a file is server-side because it "feels" like it.
// A single "use client" import chain pulls it into the browser bundle.
```

### Colocating non-route files

```
app/
  blog/
    page.tsx
    _components/Card.tsx     ← underscore: never routable
    _lib/queries.ts
```

Anything in a `_`-prefixed folder is invisible to the router, so components and helpers can live beside the route that uses them.

---

## Gotchas

| Trap | Reality |
|---|---|
| `params.slug` | `params` is a `Promise` in 16 — `await` it, or `use()` it on the client |
| `cookies().get(…)` | also a promise in 16 |
| `fetch` caching | **not** cached by default since 15; it was in 13/14 |
| `revalidateTag("x")` | needs a second argument in 16 |
| `revalidateTag` after a mutation | only marks stale; use `updateTag` for read-your-own-writes |
| missing `default.tsx` in a parallel slot | a build error in 16 |
| `middleware.ts` | renamed `proxy.ts`; the old name is deprecated |
| `error.tsx` without `"use client"` | build error — it must be a client component |
| `error.tsx` catching its sibling layout | it does not; you need the parent's boundary |
| `redirect()` inside `try/catch` | the catch swallows the control-flow throw |
| `useSearchParams` without `<Suspense>` | forces the route to client-render |
| `"use client"` in a deep file | the whole import subtree becomes client code |
| a function prop from a Server Component | runtime serialisation error |
| `process.env[key]` on the client | not inlined; `undefined` |
| a Server Function | a public endpoint — authenticate inside it |
| `cookies()` inside `"use cache"` | request-scoped data in a shared cache entry |
| `page.tsx` + `route.ts` in one folder | conflicting route error |
| an `await` above `<Suspense>` | blocks the whole page; push it below |

```tsx
// the streaming mistake, side by side
// ✗ export default async function P() {
//     const d = await slow();                       // page blocks here
//     return <Suspense fallback={<S/>}>{d}</Suspense>;
//   }
// ✓ export default function P() {
//     return <Suspense fallback={<S/>}><Slow /></Suspense>;
//   }

// the caching mistake
// ✗ "use cache" + cookies()   → the first user's cookie is served to everyone
// ✓ read cookies() in the caller, pass the value as an argument

// the boundary mistake
// ✗ app/_lib/db.ts imported by a "use client" component
//   → your database driver is now in the browser bundle (or the build fails)
// ✓ import "server-only" at the top of db.ts so the failure is immediate
```

---

## Next 15 → 16

| Area | Next 15 | Next 16 |
|---|---|---|
| `params` / `searchParams` | promise, sync access deprecated | promise, sync access **removed** |
| `cookies()` / `headers()` / `draftMode()` | same | same |
| bundler | webpack default | **Turbopack default**, webpack removed |
| `middleware.ts` | the convention | `proxy.ts`, exporting `proxy` |
| partial pre-rendering | `experimental.ppr` | `cacheComponents: true` |
| `revalidateTag(tag)` | one argument | two — a profile is required |
| read-your-own-writes | not available | `updateTag()` |
| client cache refresh | `router.refresh()` | also `refresh()` from `next/cache` |
| React Compiler | experimental | stable, `reactCompiler: true` |
| parallel route slots | `default.tsx` optional | **required** |
| `next lint` | available | removed — run ESLint directly |
| linting during `next build` | ran | no longer runs |
| `serverRuntimeConfig` / `publicRuntimeConfig` | available | removed — use `.env` |
| AMP | supported | removed entirely |
| `images.minimumCacheTTL` | 60 s | 4 hours |
| `images.qualities` | any value | `[75]`; others error |
| `images.imageSizes` | included 16 | 16 removed |
| `unstable_after` | behind a flag | `after`, stable |
| `unstable_noStore` | the escape hatch | `connection()` |
| React version | 19.0 | 19.2+ — `Activity`, `useEffectEvent`, `ViewTransition` |
| `typedRoutes` | experimental | stable |
| upgrading | manual version matching | `next upgrade` (16.1+) |

```bash
npx @next/codemod@canary upgrade latest     # runs the codemods
npx @next/codemod@canary next-async-request-api .
npx @next/codemod@canary middleware-to-proxy .
```

Node.js 20.9+ and TypeScript 5.1+ are the minimums for 16.

### App Router vs Pages Router

The Pages Router still works in 16 and the two can coexist in one project. The mapping:

| `pages/` | `app/` |
|---|---|
| `pages/index.tsx` | `app/page.tsx` |
| `pages/blog/[slug].tsx` | `app/blog/[slug]/page.tsx` |
| `pages/_app.tsx` | `app/layout.tsx` |
| `pages/_document.tsx` | `app/layout.tsx` (renders `<html>`/`<body>`) |
| `pages/404.tsx` | `app/not-found.tsx` |
| `pages/_error.tsx` | `app/error.tsx` |
| `pages/api/x.ts` | `app/api/x/route.ts` |
| `getServerSideProps` | `await` in the component |
| `getStaticProps` | `await` + `"use cache"` |
| `getStaticPaths` | `generateStaticParams` |
| `getInitialProps` | removed |
| `next/router` | `next/navigation` |
| `next/head` | the `metadata` export |
| `NextApiRequest` / `NextApiResponse` | `Request` / `Response` |
| `AppProps`, `NextPage`, `GetServerSideProps` types | `PageProps`, `LayoutProps`, `RouteContext` |

```tsx
// pages/ types, for a project that still has them
import type { NextPage, GetServerSideProps, InferGetServerSidePropsType } from "next";

type Props = { title: string };

export const getServerSideProps: GetServerSideProps<Props> = async (ctx) => {
  const slug = ctx.params?.slug as string;
  return { props: { title: slug } };
};

const Page: NextPage<InferGetServerSidePropsType<typeof getServerSideProps>> = ({ title }) => (
  <h1>{title}</h1>
);
export default Page;
```

---

## Version notes

| Feature | Requires |
|---|---|
| App Router (stable) | 13.4 |
| Server Actions (stable) | 14 |
| `next.config.ts` | 15 |
| async `params` / `searchParams` / `cookies` / `headers` | 15 (deprecated sync), 16 (removed) |
| `fetch` uncached by default | 15 |
| `instrumentation.ts` (stable) | 15 |
| `forbidden()` / `unauthorized()` | 15 (`authInterrupts` flag) |
| `next/form` | 15 |
| Turbopack default for dev and build | 16 |
| `proxy.ts` replacing `middleware.ts` | 16 |
| `cacheComponents` replacing `experimental.ppr` | 16 |
| `"use cache"`, `cacheLife`, `cacheTag` | 16 |
| `updateTag()`, `refresh()` | 16 |
| `revalidateTag` second argument required | 16 |
| `typedRoutes` stable | 16 |
| React Compiler stable | 16 |
| `default.tsx` required in parallel slots | 16 |
| `after`, `connection` stable | 16 |
| `next lint`, AMP, runtime configs removed | 16 |
| Turbopack filesystem caching on by default | 16.1 |
| `next upgrade` CLI | 16.1 |
| `ImageResponse` 2–20x faster | 16.2 |
| bundled docs for coding agents | 16.2 |
| MCP build-diagnostics server, docs as Markdown | 16.3 |

Next 16 shipped October 2025. Minimums: Node.js 20.9+, TypeScript 5.1+, React 19.2+.

Check with `npx next --version`.
