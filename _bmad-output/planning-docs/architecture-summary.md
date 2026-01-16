# Architecture Summary - Quick Reference

**Full Document**: `ARCHITECTURE.md` (this folder)

---

## Tech Stack

### Frontend

- **Framework**: Next.js 16.0.1 (App Router)
- **UI Library**: React 19.2.0
- **Styling**: Tailwind CSS 4.1.17
- **Components**: Radix UI (shadcn/ui)
- **Forms**: React Hook Form + Zod
- **Language**: TypeScript 5.9.3 (strict)

### Backend

- **Database**: Convex 1.28.2 (serverless, real-time)
- **Auth**: Clerk 6.34.5
- **Payments**: Stripe 19.3.0

### Development

- **Testing**: Vitest 4.0.8
- **Package Manager**: Yarn 4.10.3
- **Node**: 24.10.0 (Volta)

---

## Design Philosophy

1. **Serverless-First** - Minimize infrastructure management
2. **Real-Time by Default** - Live data updates without polling
3. **Type-Safe End-to-End** - TypeScript from database to UI
4. **Progressive Enhancement** - Works with mocks, scales to production
5. **Separation of Concerns** - Clear boundaries between layers

---

## System Layers

```
┌─────────────────────┐
│   Client Layer      │ Browser, React
└──────────┬──────────┘
           │ HTTPS / WebSocket
┌──────────┴──────────┐
│   Next.js Layer     │ Server Components, API Routes
└──────────┬──────────┘
           │ Convex SDK
┌──────────┴──────────┐
│   Backend Services  │ Convex, Clerk, Stripe
└─────────────────────┘
```

---

## Key Architecture Decisions

### Why Convex?

- Real-time subscriptions built-in
- Type-safe end-to-end (generated types)
- Serverless (zero infrastructure)
- ACID transactions
- No ORM needed

**See**: `_bmad-output/architecture-decisions/001-convex-as-backend.md`

### Why Next.js App Router?

- Server Components reduce client JS
- Streaming SSR for faster loads
- Layouts for code organization
- SEO-friendly server rendering

**See**: `_bmad-output/architecture-decisions/002-nextjs-app-router.md`

### Why Clerk?

- Drop-in authentication
- User management built-in
- OAuth providers supported
- Webhooks for user sync

---

## Core Patterns

### Server Component (Default)

```typescript
// No 'use client' - server by default
export default async function CoursesPage() {
  const { userId } = await auth();
  return <div>Courses</div>;
}
```

### Client Component (When Needed)

```typescript
'use client';
import { useQuery } from 'convex/react';
import { api } from '@/convex/_generated/api';

export function CourseList() {
  const courses = useQuery(api.courses.list);
  return <div>{/* Interactive UI */}</div>;
}
```

### Convex Backend

```typescript
// convex/courses.ts
export const list = query({
	handler: async ctx => {
		return await ctx.db.query('courses').collect();
	},
});

export const create = mutation({
	args: { title: v.string() },
	handler: async (ctx, args) => {
		const identity = await ctx.auth.getUserIdentity();
		if (!identity) throw new Error('Unauthorized');
		return await ctx.db.insert('courses', args);
	},
});
```

---

## Security Architecture

- **Authentication**: Clerk (JWT tokens)
- **Authorization**: Role-based (`isInstructor`, `isAdmin`)
- **Data Validation**: Zod schemas (frontend + backend)
- **Payment Security**: Stripe webhooks with signature verification
- **HIPAA Considerations**: No PHI stored, instructor credentials encrypted

---

## Performance Considerations

- **Server Components**: Reduce client JS bundle
- **Real-Time**: Convex subscriptions (no polling)
- **Image Optimization**: Next.js Image component
- **Code Splitting**: Automatic with App Router
- **Caching**: Convex client caching + Next.js caching

---

## Scalability

- **Serverless**: Auto-scales with traffic
- **Database**: Convex handles scaling
- **CDN**: Vercel Edge Network
- **No Servers**: Zero infrastructure to manage

---

**Full Details**: Read `ARCHITECTURE.md` (this folder) for complete system design, integration patterns, and deployment architecture.
