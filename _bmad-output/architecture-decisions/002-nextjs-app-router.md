# 002. Next.js App Router

**Date**: 2025-10-01
**Status**: Accepted
**Deciders**: Product Team, Engineering Team

---

## Context

Doctrina LMS needed a modern React framework that could:
- Render server-side for SEO (course listings, public pages)
- Support client-side interactivity (dashboards, real-time updates)
- Provide excellent performance (image optimization, code splitting)
- Scale with the product (SSR, SSG, ISR)
- Have strong TypeScript support

Next.js offers two routing systems:
1. **Pages Router** (legacy, stable)
2. **App Router** (new, React Server Components)

## Decision

We chose **Next.js 16 with App Router** (not Pages Router).

The App Router provides:
- **React Server Components** (RSC) by default
- **Streaming SSR** for faster initial page loads
- **Layouts and Nested Routing** for better code organization
- **Server Actions** for form submissions
- **Parallel and Intercepting Routes** for advanced UX

## Rationale

### Why App Router Over Pages Router

#### Server Components by Default

**Benefit**: Most components don't need client-side JavaScript
- Course listings → Server Component (no JS)
- Dashboard data fetching → Server Component (no JS)
- Forms, interactive UI → Client Component ('use client')

**Result**: Smaller client bundles, faster page loads

#### Streaming and Suspense

**Benefit**: Progressive page rendering
- Show UI shell immediately
- Stream in data as it loads
- Better perceived performance

**Pattern**:
```typescript
// app/courses/page.tsx
import { Suspense } from 'react';

export default function CoursesPage() {
  return (
    <div>
      <Suspense fallback={<LoadingSkeleton />}>
        <CourseList />
      </Suspense>
    </div>
  );
}
```

#### Layouts

**Benefit**: Shared layouts without duplicating code
- Dashboard layout with sidebar
- Course layout with tabs
- Auth layout centered

**Pattern**:
```typescript
// app/(dashboard)/layout.tsx
export default function DashboardLayout({ children }) {
  return (
    <div className="flex">
      <Sidebar />
      <main>{children}</main>
    </div>
  );
}
```

---

## Consequences

### Positive

1. **Better Performance**: Server Components reduce client JavaScript
2. **SEO-Friendly**: Server-rendered course pages for search engines
3. **Future-Proof**: App Router is the future of Next.js
4. **Better DX**: Layouts, parallel routes, intercepting routes
5. **Streaming SSR**: Faster perceived performance

### Negative

1. **Learning Curve**: Team must learn Server Components mental model
2. **Debugging Complexity**: Server/client boundary can be confusing
3. **Third-Party Libraries**: Some libraries don't support RSC yet
4. **Breaking Changes**: App Router still evolving (Next.js 16 is recent)
5. **Migration Pain**: Migrating from Pages Router would be difficult

---

## Implementation Patterns

### Server Component (Default)

```typescript
// app/courses/page.tsx
import { auth } from '@clerk/nextjs/server';

export default async function CoursesPage() {
  const { userId } = await auth();

  return (
    <div>
      <h1>Courses</h1>
      {/* Server-rendered, no client JS */}
    </div>
  );
}
```

### Client Component (When Needed)

```typescript
// components/CourseCard.tsx
'use client';

import { useState } from 'react';

export function CourseCard() {
  const [liked, setLiked] = useState(false);

  return (
    <div>
      <button onClick={() => setLiked(!liked)}>
        {liked ? 'Unlike' : 'Like'}
      </button>
    </div>
  );
}
```

### Mixing Server and Client

```typescript
// app/courses/page.tsx (Server Component)
import { CourseCard } from '@/components/CourseCard'; // Client Component

export default async function CoursesPage() {
  const courses = await getCourses(); // Server-side

  return (
    <div>
      {courses.map(course => (
        <CourseCard key={course.id} course={course} />
      ))}
    </div>
  );
}
```

---

## Alternatives Considered

### Alternative 1: Next.js Pages Router

**Pros**:
- More stable (battle-tested)
- Larger ecosystem (more examples)
- Simpler mental model (no RSC)
- Easier debugging

**Cons**:
- More client JavaScript (no Server Components)
- No streaming SSR
- No layouts (must use custom _app.tsx patterns)
- Not the future of Next.js

**Why Not**: App Router's performance benefits and future-proofing outweigh Pages Router's stability.

### Alternative 2: Remix

**Pros**:
- Excellent data loading (loaders/actions)
- Built-in form handling
- Nested routing
- Fast page transitions

**Cons**:
- Smaller ecosystem than Next.js
- No Server Components yet (coming)
- Less mature image optimization
- Fewer deployment options

**Why Not**: Next.js has larger ecosystem and better Vercel integration (our hosting).

### Alternative 3: Vite + React Router

**Pros**:
- Full control over architecture
- No framework lock-in
- Fast development server (Vite)
- Flexible routing

**Cons**:
- No SSR out of the box
- Must build SEO solution manually
- No built-in image optimization
- More setup required

**Why Not**: Next.js provides SSR, SEO, image optimization out of the box. No need to reinvent the wheel.

---

## Migration Considerations

### If We Need to Revert to Pages Router

**Likely Reasons**:
- App Router bugs become blockers
- Third-party libraries don't support RSC
- Team struggles with Server Components mental model

**Migration Path**:
1. Move `app/` files to `pages/`
2. Convert Server Components to `getServerSideProps` or `getStaticProps`
3. Remove 'use client' directives (all components are client by default)
4. Rewrite layouts using `_app.tsx` and `_document.tsx`
5. Update routing patterns

**Estimated Effort**: 2-3 weeks

**Risk**: Low. Next.js supports both routers side-by-side during migration.

---

## Key Guidelines

### When to Use Server Components

✅ **Use Server Components (default)** for:
- Data fetching from Convex
- Layouts and static UI
- SEO-critical pages (course listings, landing pages)
- Authentication checks

### When to Use Client Components

✅ **Use Client Components ('use client')** for:
- Interactivity (buttons, forms, modals)
- Browser APIs (window, localStorage, WebSocket)
- React hooks (useState, useEffect, useContext)
- Third-party libraries requiring client-side

### Guidelines

1. **Start with Server Component**: Default to server, add 'use client' only when needed
2. **Minimize Client Bundles**: Keep 'use client' components small and focused
3. **Pass Data Down**: Server Components can pass data to Client Components as props
4. **No Server-Only Code in Client**: Don't import database code into Client Components

---

## References

- [Next.js App Router Documentation](https://nextjs.org/docs/app)
- [React Server Components](https://react.dev/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#react-server-components)
- [Next.js 16 Release Notes](https://nextjs.org/blog/next-16)
- Project File: `docs/ARCHITECTURE.md` (Next.js Layer section)
- Project File: `.claude/standards/nextjs.md` (Next.js patterns)
- Project File: `.claude/standards/react.md` (React patterns)

---

**Last Updated**: 2026-01-15
**Next Review**: After Next.js 17 release
