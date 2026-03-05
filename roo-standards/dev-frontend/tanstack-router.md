## // TanStack Router Configuration & Usage Rules

name: tanstack_router
description: Core concepts and implementation rules for TanStack Router (File-Based)
globs: ["src/routes/**/*.{ts,tsx}", "src/app.tsx", "src/main.tsx"]
tags:

- react
- routing
- typescript
- state-management

---

# TanStack Router v1

## Core Routing Concepts

- **File-Based Routing**: The filesystem defines your route hierarchy.
- **`createFileRoute`**: Used for all routes except the root. It requires the path string for TypeScript's "magic" type safety.
- _Note_: This path is automatically managed by the TanStack Router Bundler Plugin.

- **The Root Route**: The entry point of your route tree.
- Use `createRootRoute()` for standard setups.
- Use `createRootRouteWithContext<T>()` to inject dependencies like TanStack Query's `QueryClient`.

### The Component Tree

- **`<Outlet />`**: The placeholder component where child routes render.
- **Default Behavior**: If a route has no `component` defined, it defaults to rendering an `<Outlet />`.

---

## Route File Patterns

TanStack Router uses a flexible naming convention to handle nesting, layouts, and exclusions.

| Pattern        | Filename Example      | Resulting Path        | Description                                          |
| -------------- | --------------------- | --------------------- | ---------------------------------------------------- |
| **Index**      | `posts.index.tsx`     | `/posts`              | Matches parent exactly; no children.                 |
| **Dynamic**    | `posts.$postId.tsx`   | `/posts/123`          | Captures segment into `params`.                      |
| **Optional**   | `{-$lang}.about.tsx`  | `/about`, `/en/about` | Optional path segments.                              |
| **Splat**      | `files.$.tsx`         | `/files/a/b/c`        | Matches all remaining segments (`_splat`).           |
| **Pathless**   | `_layout.tsx`         | (None)                | Wraps children in a component without a URL segment. |
| **Non-Nested** | `posts_.$id.edit.tsx` | `/posts/123/edit`     | Un-nests from parent layout (`posts.tsx`).           |
| **Excluded**   | `-components/btn.tsx` | (None)                | Prefixed with `-` to be ignored by the router.       |

### Path Param Enhancements

- **Prefixes/Suffixes**: Use curly braces for complex matching.
- `post-{$id}.tsx` matches `/post-123`.
- `{$file}.txt.tsx` matches `/report.txt`.

---

## Navigation & Type Safety

### The `<Link>` Component

Always preferred over imperative navigation for `<a>` tag benefits (SEO, cmd+click).

```tsx
<Link
  to="/posts/$postId"
  params={{ postId: '123' }}
  search={{ view: 'compact' }}
  hash="comments"
  activeProps={{ className: 'font-bold' }}
>
  View Post
</Link>
```

### Relative Navigation

- **`.`**: Reloads the current route or navigates relative to the `from` prop.
- **`..`**: Navigates to the parent route.
- **`from`**: Always specify the origin route for full TypeScript autocomplete on relative paths.

### Imperative Navigation

Use `useNavigate()` for side-effects (e.g., redirecting after a form submission).

> ⚠️ **Caution**: `router.navigate` is available globally, but `useNavigate` is the hook-based standard for React components.

---

## Search Param Management

TanStack Router treats the URL as **JSON-first state**.

### Validation with Zod

Use the `zodValidator` adapter for clean, type-safe search params.

```tsx
import { z } from 'zod';
import { zodValidator } from '@tanstack/zod-adapter';

const searchSchema = z.object({
  page: z.number().catch(1),
  filter: z.string().optional(),
});

export const Route = createFileRoute('/shop')({
  validateSearch: zodValidator(searchSchema),
});
```

### Search Middlewares

Modify search params globally or per-route before the URL is generated.

- `retainSearchParams(['token'])`: Keeps the `token` param across all navigations.
- `stripSearchParams({ page: 1 })`: Removes `page=1` from the URL if it matches the default.

---

## Accessing Data & Params

- **`useParams()`**: Access dynamic segments (e.g., `{ postId: string }`).
- **`useSearch()`**: Access validated search params.
- **`useLoaderData()`**: Access data returned from the route's `loader`.
- **`getRouteApi(path)`**: Access typed hooks in code-split files without importing the `Route` object.

```tsx
const routeApi = getRouteApi('/posts/$postId');

function PostComponent() {
  const { postId } = routeApi.useParams();
  const data = routeApi.useLoaderData();
  return <div>{data.title}</div>;
}
```
