---
name: tanstack-start
description: TanStack Start framework patterns and best practices. Use when writing or modifying TanStack Start code — routes, server functions, middleware, SSR configuration, or execution model questions.
user-invocable: true
---

# TanStack Start Reference

TanStack Start is a full-stack React framework powered by TanStack Router and Vite. It provides full-document SSR, streaming, server functions, bundling, and universal deployment. It is currently in Release Candidate stage.

## What TanStack Start is

- A full-stack React framework built on **TanStack Router** (type-safe routing) + **Vite** (build tooling)
- Provides: SSR, streaming, server functions (type-safe RPC), server routes (API endpoints), middleware, full-stack bundling
- Does NOT currently support React Server Components (planned)
- Deploy to any Vite-compatible host (Vercel, Netlify, Cloudflare, Node.js, Bun, static)

## Project structure

```
src/
├── router.tsx           # Router config — exports getRouter(), creates router with routeTree
├── start.ts             # (optional) Global middleware, createStart() config
├── routeTree.gen.ts     # Auto-generated route tree — never edit, regenerated on dev/build
└── routes/
    ├── __root.tsx        # Root layout — <html>, <head>, <body>, HeadContent, Scripts, Outlet
    ├── index.tsx         # "/" route
    ├── about.tsx         # "/about" route
    ├── posts.tsx         # "/posts" layout route (renders Outlet for children)
    └── posts.$postId.tsx # "/posts/:postId" dynamic route
vite.config.ts           # Vite config with tanstackStart() plugin
```

## Key patterns

### Route + server function

```tsx
// src/routes/index.tsx
import { createFileRoute } from '@tanstack/react-router'
import { createServerFn } from '@tanstack/react-start'

const getData = createServerFn({ method: 'GET' }).handler(async () => {
  // Runs on server only — safe to use secrets, DB, filesystem
  return { count: 42 }
})

export const Route = createFileRoute('/')({
  loader: () => getData(),   // Loader is isomorphic — calls server fn which handles the boundary
  component: Home,
})

function Home() {
  const data = Route.useLoaderData()
  return <div>Count: {data.count}</div>
}
```

### Server route (API endpoint)

```ts
// src/routes/api/health.ts
import { createFileRoute } from '@tanstack/react-router'

export const Route = createFileRoute('/api/health')({
  server: {
    handlers: {
      GET: async () => Response.json({ status: 'ok' }),
    },
  },
})
```

## Detailed reference files

Load the relevant file(s) from the `reference/` directory based on what the user is working on:

- **[reference/execution-model.md](reference/execution-model.md)** — Core execution model: isomorphic-by-default principle, server vs client environments, execution control APIs (`createServerFn`, `createServerOnlyFn`, `createClientOnlyFn`, `createIsomorphicFn`, `ClientOnly`, `useHydrated`), architecture decision framework, and security considerations.

- **[reference/code-execution-patterns.md](reference/code-execution-patterns.md)** — Practical patterns for controlling where code runs: progressive enhancement, environment-aware storage, common problems (env variable exposure, incorrect loader assumptions, hydration mismatches), and production checklist.

- **[reference/server-and-env-functions.md](reference/server-and-env-functions.md)** — Server functions (`createServerFn`) deep dive: basic usage, calling from loaders/components/hooks, file organization (`.functions.ts` / `.server.ts` conventions), parameters & validation (Zod, FormData), error handling & redirects, server context utilities (`getRequest`, `setResponseHeaders`, etc.), and environment functions (`createIsomorphicFn`, `createServerOnlyFn`, `createClientOnlyFn`).

- **[reference/server-routes.md](reference/server-routes.md)** — Server route API endpoints: defining handlers (simple and with middleware), file route conventions, handler context (`request`, `params`, `context`), dynamic path params, wildcard/splat params, request bodies, JSON responses, status codes, and response headers.

- **[reference/middleware.md](reference/middleware.md)** — Middleware system: request middleware vs server function middleware, composition, `next()` chain, context management (`sendContext` client↔server), global middleware (`src/start.ts` with `createStart`), execution order, header merging, custom fetch, and environment tree shaking.

- **[reference/router.md](reference/router.md)** — Routing fundamentals: `router.tsx` setup, file-based routing in `src/routes/`, root route (`__root.tsx`), `HeadContent`/`Outlet`/`Scripts` components, route tree generation, nested routing, route types (index, dynamic, pathless layout, non-nested, grouped), and `createFileRoute`.

- **[reference/optional-ssr.md](reference/optional-ssr.md)** — Selective SSR: `ssr: true | false | 'data-only'` per-route configuration, functional form for runtime decisions, inheritance rules (child can only be more restrictive), fallback rendering with `pendingComponent`, disabling root route SSR with `shellComponent`, and `defaultSsr` in `createStart`.

- **[reference/hydration-errors.md](reference/hydration-errors.md)** — Debugging hydration mismatches: common causes (Intl, Date, random IDs), strategies for making server/client match, locale/timezone middleware patterns.

- **[reference/error-boundaries.md](reference/error-boundaries.md)** — Error boundary setup: route-level `errorComponent`, root error handling, error recovery patterns.

- **[reference/authentication.md](reference/authentication.md)** — Auth patterns: session management, protected routes via `beforeLoad`, redirect-based auth guards, cookie handling, auth middleware composition.

- **[reference/server-entry-point.md](reference/server-entry-point.md)** — Customizing `src/server.ts`: `createStartHandler`, custom server setup, request handling pipeline.

- **[reference/spa-mode.md](reference/spa-mode.md)** — Full SPA mode: disabling all SSR, client-only rendering, when to use SPA vs selective SSR.

- **[reference/static-prerendering.md](reference/static-prerendering.md)** — Static site generation: prerendering routes at build time, configuration options.

- **[reference/environment-variables.md](reference/environment-variables.md)** — Env var handling: `VITE_` prefix for client exposure, server-only secrets, `.env` file patterns, runtime vs build-time variables.

- **[reference/import-protection.md](reference/import-protection.md)** — Bundle safety: preventing server code from leaking into client bundles, import boundaries, tree shaking behavior.

- **[reference/hosting.md](reference/hosting.md)** — Deployment targets and adapters: Vercel, Netlify, Cloudflare, Node.js, Bun, static hosting configuration.

- **[reference/databases.md](reference/databases.md)** — Database integration patterns: connecting from server functions, ORM usage, connection pooling.

- **[reference/path-aliases.md](reference/path-aliases.md)** — Path alias configuration: `~` imports, tsconfig paths setup.

## Key principles to always apply

1. **Loaders are isomorphic** — they run on BOTH server and client. Never put secrets in loaders directly; use `createServerFn()` instead.
2. **Server functions are RPC** — `createServerFn()` runs on server but is callable from anywhere. The build replaces implementations with fetch stubs in client bundles. Static imports of server functions in client code are safe.
3. **`createServerOnlyFn` vs `createServerFn`** — `createServerOnlyFn` throws on client (for utilities only used server-side). `createServerFn` creates an RPC endpoint (callable from client via network request).
4. **Middleware is composable** — request middleware for all requests, server function middleware for server fns only. Both support `next()` chaining and context passing. Global middleware goes in `src/start.ts` via `createStart()`.
5. **File-based routing** — routes in `src/routes/` map to URL paths. Server routes use `.ts`, page routes use `.tsx`. Same file can have both `server.handlers` and `component`.
6. **Route tree is auto-generated** — `routeTree.gen.ts` and the path string in `createFileRoute()` are managed by the bundler plugin. Never edit them manually.
7. **Environment variables** — only `VITE_` prefixed vars are exposed to client bundles. All other env vars are server-only. Use `createServerFn()` or `createServerOnlyFn()` to access secrets.
8. **SSR is on by default** — every route SSRs unless explicitly opted out with `ssr: false` or `ssr: 'data-only'` per-route, or `defaultSsr: false` globally in `createStart()`.
9. **Avoid dynamic imports of server functions** — always use static imports. Dynamic `import()` of server functions can cause bundler issues.
